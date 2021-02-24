---
title: CRTE Bootcamp Review
layout: single
categories: [blog]
permalink: /:categories/:year/:month/:day/:title/
classes: wide
toc: true
show_date: true
---

After a friend of mine went through CRTP, and my Offshore experience, I felt like I wanted to go further with windows and AD stuff in general, and consolidate what I had learnt during Offshore. That was about the time PentesterAcademy announced they were launching bootcamps, the first for AD beginners was in December, and they would have an advanced AD bootcamp centred on CRTE in January, so I jumped on it, along with my friend.

# Bootcamp

The bootcamp consisted of 4 sessions of 3 hours on Zoom with [Nikhil](https://twitter.com/nikhil_mitt), once per Sunday evening in the CET time zone, along with slides, and recordings of the live sessions if it's too late where you live. Additionally, we all got access to a Discord server managed by Nikhil.

It started pretty easy, and probably a good deal of overlap with CRTP, but quickly went into Exchange, MSSQL, and Azure related attacks, plus a lot of cross-domain and cross-forest specific attacks. Overall the materials were very good and well explained, and having live sessions and access to the Discord server meant we could ask questions and have Nikhil answer them pretty quickly, and have some connection to the other people taking the bootcamp. The 3-hours sessions were long and strenuous sometimes, but having the recording meant we could go through them with the slides again at our own pace along with the lab during the week, which was very nice. Also, we got to see Nikhil drawing live:

![Nikhil explaining database links]({{site.url}}/assets/images/nikhil-drawing.png)

# Labs

We also had 30 days of access to the CRTE labs (or something that resembles the CRTE labs a lot), where we could implement the enumeration and abuses we were learning during the bootcamp. We received a full lab manual, and Nikhil later released a Covenant lab manual, which was great to get to know this C2 better. Nikhil spent a lot of time during the sessions emphasizing OPSEC and how to avoid logging and detection, which was great and covered some nice tools and new ways to bypass. Obviously, I forgot all of it and was noisy **AF** in my exam.

At the end of the labs, we had gone through several domains and forests, trusts and links all around them, and the paths we took looked like that: (blurred to hide machine names and actual attacks)

![CRTE labs attack paths]({{site.url}}/assets/images/crte-labs.png)

Having such a detailed lab manual was a blessing and a curse at the same time though, it was easy to refer to it or the slides for commands, tools, or particular abuses, but I procrastinated doing the labs a lot. Comparing with my Offshore experience, where there was 0 lab manual, I think I was not used to being spoon-fed the lab, and wanted to have to try harder, had to fight that a bit. In the end I completed the whole lab the manual way, and did it again on the last few days of our lab access using Covenant, which taught me a lot I could have used during Offshore!

# Exam

The exam was a 48 hours hands-on exam, with 6 machines (when I took it), of which we had to attain remote code execution on at least 3. I actually stressed myself timewise a bit more than I had to as I thought we had 48 hours to do the exam and write the report, well, turns out, it was 48 hours for the exam, and 48 hours more to write the report! But well, on the other hand I got a nice weekend.

I started my exam the day after our lab access ended early in the morning, and thus began at least 30 hours of sufferance through 6 machines and a few domains. Without saying too much, there were stuff we saw in the bootcamp, some stuff we didn't and had to learn during the exam, and a lot of moments I was banging my head on the desk. Any sense of OPSEC flew out of the window to be completely honest, any blue teamer on the other side would have seen me coming a few km away, and I focused on being able to own all machines before the end of the 48 hours. But at last with an evening to spare before sending the report, all machines owned, I could finally rest, the night had been short.

# Review

Overall, I would say that having live sessions was a very nice bonus to normal courses, and Nikhil could answer everything we threw at him. The normal and Covenant lab manuals were a good guide for the labs as well, and Nikhil showed most attacks and abuses in the sessions.

I would recommend the CRTE bootcamp (if Nikhil continues it in the future) to anyone familiar with AD, but wanting to go further and consolidate their knowledge of AD attacks. There was probably a good deal of overlap with CRTP material, but from what others said who took both, it was still worth it as it went much further in some areas, introduced new concepts, and was more focused on OPSEC. **BUT**, if you're more the type to learn along the way, opt for the normal CRTE lab+exam instead. It really depends on your learning type. Otherwise, future bootcamps are [here](https://bootcamps.pentesteracademy.com/)!
