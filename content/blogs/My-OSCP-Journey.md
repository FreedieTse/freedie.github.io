+++
date = '2026-05-30T15:09:26-04:00'
draft = false
title = 'From Failing with 0 Points to 100 Points in 7 Hours'
tags = ["oscp", "certifications", "offsec"]
+++

# Background
For context, **I have no prior work experience to pentesting** or cybersecurity other than my CompTIA Secucirty+ certification, but I have some programming background mostly in Java, C, C++ and Python. I am graduating in June of 2026 from University of Waterloo with a major in Mathematics and a minor in computing (basically computer science). I got my **CompTIA Security+** in mid September, 2025 after studying for two weeks. Then completed my **Cisco CCNA** in late October of 2025 after studying for a month. After earning my Cisco CCNA, I started studying for my school final exams, and finally **I started officially studying the PWK 200 course in late December, 2025.**

---
# Fails
I booked my first OSCP attempt on Janurary 16th, 2026. **To prepare well for OSCP**, *we must “reconnaissance” on the exam structure*: I first researched about the current OSCP exam on reddit, appearantly the intermediate to hard level Proving Grounds Practice (platform by OffSec, also known as PGP) labs from [LainKusanagi’s OSCP Like List](https://docs.google.com/spreadsheets/d/18weuz_Eeynr6sXFQ87Cd5F0slOj9Z6rt/) was the closest to actual OSCP. *Sections like AWS, Antivirus Evasions were not part of the exam*. Analyzing the OSCP exam: **to pass the exam I need 70 out of 100 points**. The OSCP exam has an assumed breach Active Directory (AD) set with one given credentials that is worth 40 marks in total, and 3 stand-alones that I initially thought were all linux for some reason, each worth 20 points. **So to pass the exam, I needed at least 10 points on AD and root all of the stand-alones, or root the whole AD set and get 30 points split among the stand-alones**.
![](https://manage.offsec.com/wp-content/uploads/2023/02/image1-1.png)
My strategy when preparing for the first OSCP exam was **simple**: go over all of the PWK 200 course reading that was relevant to the exam (**skipping the irrelevant sections**) and add the methodologies that I saw in to my methodology, but **I recommend spending more time on the course content and labs**. Then after going over the PWK 200 course quickly, I started getting hands-on experience from of all the easy to hard labs of PGP, and refining my methodology. I rushed through around 4 to 5 labs labs per day, which was not a good idea. **However, I learned that looking at walkthroughs is not bad at all, but you need to try everything you know first. Learn and digest what you were lacking from.**

On my first OSCP exam attempt: I got local admin on the first Windows machine in the AD set within 10 minutes, but got stuck for 4 hours before trying the stand-alones and also got initial access on the first stand-alone in 10 minutes. So now, I knew to pass the exam I had to either get root on the first stand-alone, or get more points on the AD set. However, I never got further. I **failed the first attempt**, with *only 20 points*.

![](https://cdn.shopify.com/s/files/1/0306/6419/6141/files/Copy_of_fail_1024x1024.png?v=1617976957)

**Surprisingly**, I was excited after my failure from my first attempt. **I know that there is so much more that I can learn**. Then I arranged my second attempt on February 22nd, 2026. This time, **I finished all of the Proving Grounds Practice labs on LainKusanagi’s List**, and took detailed notes over them while adding to my methodology. To prepare better for AD: I completed most of the AD boxes from Hack The Box on LainKusanagi’s List and also watching over the walkthroughs of AD Chains from Hacker Blueprint. In addition, I skimmed through [0xdf writeups](https://0xdf.gitlab.io/) of all of the Hack The Box machines from LainKusanagi’s List, and watching some of [IppSec](https://www.youtube.com/@ippsec/videos) videos.

Honestly, looking back from now, I had enough skills to pass the OSCP exam. But **I failed, *with 0 points***. This was mainly because I reached the “***cybersecurity burnout***” due to the massive amount of training. When I was doing the labs near end of my preparation, I **stopped actively thinking** and hoping that they had simple exploits; when my insight did not work, I was not able to pivot to another strategy, but rather getting depressed. I never thought this would happen to me, but **I know that I needed to take a break**. I went on a trip for 2 weeks, doing absolute nothing from cyber security.

---
# Acing the OSCP
I started studying for the OSCP again after finishing my last final on April 26th. This time I have arranged my OSCP exam on May the 26th, leaving my self a little more time to practice for the exam. First thing that I did, which I spend 3 days on that saved me so much time on the exam: **I organized my notes** to a more structued and logical workflow. For example, for AD and Windows enumeration, I ordered a workflow starting with:

- **Unauthenticated**: such as sync my kali time with the AD DC, adding target IPs and host names into `/etc/hosts`, NTLM capture, etc.
- **Authenticated without Shell**: such as make an `ips` list file to enumerate multiple host with `netexec`, enumerate open ports, etc.
- **Shell with Low Privilege User**: such as pivoting with `ligolo-ng`, trying to do local privilege escalation, or search for credentials, etc.
- **After Privilege Escalation**: add a user as admin and then dump credentials with `netexec`, search for credentials, etc.

The new structured notes helped me *visualize* my methodology in a more **logical workflow** such that: I know if I couldn’t get further access after trying my methodology twice or three times, then it might not be something that I missed and I should start getting **creative**.
![](https://www.netexec.wiki/~gitbook/image?url=https%3A%2F%2F361548579-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fb0qbsNvsXjRTsQcNipGM%252Fuploads%252Fgit-blob-aa8c6676dd772f8004894c5e770b8135a230244d%252FNetExec-Logo-Color.png%3Falt%3Dmedia&width=1248&dpr=3&quality=100&sign=f7085d8f&sv=2)
Since I was weak on stand-alones, I decided to practice on Virtual Hacking Labs. To be honest, **VHL** is very **underated** but quite *expensive* (*$99/month*). Even though there are some kernel exploits (rare on the OSCP) but the practice **deepened my Linux understanding** (*especially privilege escalation techniques such as utilizing different capabilities that are **not** on GTFOBins*). During this time, I **reached out to LainKusanagi via discord**, he is so kind and taught tips and tricks that are very helpful to the OSCP exam.
I also practiced on some labs from **HackSmarter** which are **very high quality**. On the other hand, the **Hacker Blueprint’s AD Chains** taught me a handful of methodologies that was **very helpful during my OSCP exam**.

On my third exam day, I tried to join exam at exam time at 1 PM but it turned out the time zone that my exam was arranged was different than my actual time zone, so my exam did not starting until 6 PM. I was really frustrated, but decided that I should save my energy until 6 PM so I layed on bed doing the minimal thinking and energy consuming thing like listening to music and taking a nap.

![](https://media.licdn.com/dms/image/v2/C4E12AQHSpA_ny4uh3w/article-cover_image-shrink_720_1280/article-cover_image-shrink_720_1280/0/1615816513305?e=2147483647&v=beta&t=oKyf7sXa1P-ZWP2-KA3HHHDrX0m2SCZVDvWtp-DH8PI)

At 6 PM I started my third attempt of OSCP: I couldn’t manage to get local admin until 1 hour in, but since I was enumerating the during that 1 hour too, **I rooted the whole AD set at 8 PM after 2 hours of starting the exam**. Then I took a break and went to get dinner.

After I was back from dinner, I rooted the first stand-alone within an hour. I encountered **unfamiliarities**, but with googling and the help of [hacktricks](https://hacktricks.wiki/en/index.html) and [hackviser](https://hackviser.com/tactics/pentesting), I was able to learn during the exam. At this point, I had 60 points already, so *I just needed 10 points to pass the exam*. However, I was fell in rabbit hole and did not find a way into the 2nd or 3rd stand-alone. **I was stuck for 2 hours** without and progress and just as I thought I might fail with 60 points, I thought I should re-enumerate all the ports again.

Somehow, **after this re-enumeration**, I found a way in stand-alone 2 **within 10 minutes** and the privilege escalation also took me under 10 minutes too. Before I went back to the stand-alone 3, **I had already passed the exam with 80 points**. Therefore I told myself I could give 100 points a try, and with a little bit of luck, I found a set of keys that allowed me to proceed to root the last stand-alone within 30 minutes after rooting the second.

---
# The Beginning
To this day I still cannot believe **I got 100 points on OSCP under 7 hours**, in which I consider pretty fast, being dramatically different from getting 0 points on the second attempt too.

Throughout this journey, I had a lot of *struggles*, but **I overcame them and became the better of me**. This is not the end of my journey as I got even more addicted to improving my skills during the journey. I will be practicing more of my skills on the Hack The Box platform from now, but I will go back to the OffSec platform soon after to learn even more. Thank you for those that have read this far from my first post!

P.S.
This post is more of a journal-style post, but I will be posting a detailed list of tips for OSCP that I think you should add to your methodology list soon. Hopefully this would help any of you preparing for the OSCP in the future!
