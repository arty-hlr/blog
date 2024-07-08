---
title: eLearnSecurity Certified eXploit Developer Review
layout: single
categories: [blog]
permalink: /:categories/:year/:month/:day/:title/
classes: wide
toc: true
show_date: true
excerpt: "A review of the XDS course/eCXD exam by eLS"
---

# Update 25.01.2021

In the year or so since this blog post, eLS has been acquired by INE, the courses are now only available as a subscription on [INE](https://ine.com/)'s website. Also, OffSec has launched a basic exploit dev course of their own [here](https://www.offensive-security.com/exp301-osed/) focused on windows, which still doesn't cover x64 though, but seems very thorough on many other aspects, if you're starting out with windows exploit dev, I now recommend going for OSED.

# Intro

Last year, I had bought vouchers for the eWAPT and eCXD exams from eLearnSecurity. I didn't get around to actually prepare or take those exams apart from some binary exploitation studying, but a few weeks ago when an email came saying I had until mid November to take those exams, this jolted me back to action, at least on the binary exploitation front. End of october was actually the beginning of lockdown in Germany, my contract with my previous work had just ended, so I was gonna have a few weeks of free time ahead of me, why not focus on XDS, finally get the windows part of the course down, and try the exam?

# Preparation

I felt like I had enough linux binary exploitation experience from CTFs, and the few HTB boxes I did that had some bin exp in them (👀 looking at you October, Safe, Ellingson, and especially Rope), and I hadn't come round to finishing to windows part of the course last november, so this was my focus. 

From OSCP I was comfortable with basic vanilla buffer overflows, debugging with [Immunity](https://www.immunityinc.com/products/debugger/), and [Mona](https://www.corelan.be/index.php/2011/07/14/mona-py-the-manual/), but I wanted to try another (more modern) debugger, so I partly switched to [x64dbg](https://x64dbg.com/), keeping Immunity handy for Mona though. (although since then I heard there was a x64dbg [fork](https://github.com/x64dbg/mona) for it!)

From the syllabus I wanted to do specifically the last windows modules:

![XDS Syllabus]({{site.url}}/blog/assets/images/xds-syllabus.png)

As I had dabbled with SEH and egghunting earlier in the year in April, but a review would be in order!

The repos I used for my exploit dev practice are [here](https://github.com/arty-hlr/exploitdev-practice) and [here](https://github.com/arty-hlr/shellcode-practice) (see my shellcode post as well). I documented the exploits I developed for all the vulnerable software I used in the windows exploit dev prep (all installed on a Winwows 7 x86 Flare VM), and the binaries are there as well if you wanna practice on them. Overall the most interesting modules were those that touched windows ROP and shellcoding, although unfortunately they were a bit shallow and only used Mona to generate the ROP chains (spoiler alert, that wouldn't be enough for the exam).

Some more resources I used along the way:

+ [Corelan](https://www.corelan.be/index.php/category/security/exploit-writing-tutorials/)
+ [Capt. Meelo](https://captmeelo.com/category/exploitdev)
+ [h0mbre](https://h0mbre.github.io/page8/)
+ [fuzzySecurity](http://www.fuzzysecurity.com/tutorials/expDev/1.html)

# Exam

With only 2 weeks of review and additional prep, I wasn't feeling 100% confident for the exam, but I wanted to get it over with, so here I went on a Wednesday at 6 PM, starting my 48 hours exam. I knew one person who had passed the exam, although not with all the points, so I wanted to see if I could. I can't say much about the exam per se, but it was easy on one hand, and required some very specific skills on the other hand. Overall, it could and should have covered more of the syllabus though.

After 40 or so hours of gruesome debugging and exploiting work, here I was with shells on all objectives. A day more on the exam report (for which I used, like for OSCP, the wonderful markdown [template](https://github.com/noraj/OSCP-Exam-Report-Template-Markdown)), and I had my weekend for me!

I sent my report on the 16th of November, it took until the 8th of December to get the email that I had passed though 😅 eLS can definitely do better with exam rating times!

# Review

Overall, the course was okay *for complete exploit dev beginners*, but it covered old techniques on old OSes, and lacked some more modern things, heap stuff, x64 windows, some more in-depth module on writing ROP chains by hand, etc. It kinda makes sense as the course is called eXploit Development *Student*, so we could expect like PTS -> PTP -> PTX maybe some more advanced exploit dev courses like XDP or XDX who knows, but so far nothing has been announced indicating that. It's unfortunately one of the only exploit dev courses out there (not counting OSCP or the very outdated OSCE), so I'm sure they can, or someone can do better in the future.

To conclude, if you're completely new to exploit dev, do take the course as it's gonna bring you some important basic knowledge for more advanced attacks. If you're already familiar with linux exploit dev, and want to transfer that knowledge to windows, probably better stick to the lots of blog posts out there (that XDS often cites) and build your own lab!
