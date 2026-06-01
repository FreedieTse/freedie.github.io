+++
date = '2026-06-01T00:50:23-04:00'
title = 'OSCP Tips and Tricks'
tags = ["oscp", "certifications", "offsec"]
+++

Hello! I am sure you want to ACE the OSCP like I did! And even beat my personal record of getting 100 on OSCP in under 7 hours, and I truly believe any one of you have the potential to smash the OSCP exam harder and better than I did. 

I am sure there are very good guides and tricks out there already and I will try to cover some of the tips and tricks that I haven't seen or found very important throughout my OSCP journey, hope this will help you pass the OSCP exam!

This blog may be very lengthy, for your convenience, this is the list of contents:

[Resources](#resources)
- [Proving Grounds Practice](#proving-grounds-practice)
- [Hacker Blueprint AD Chains](#hacker-blueprint-ad-chains)
- [Virtual Hacking Labs](#virtual-hacking-labs)
- [Hack Smarter](#hack-smarter)
- [Hack The Box](#hack-the-box)
- [Proving Grounds Play](#proving-grounds-play)
- [PEN-200 Challenge Labs](#pen-200-challenge-labs)

[Preparation Tips](#preparation-tips)
- [Before Doing Labs](#before-doing-labs)
- [While You are Doing Labs](#while-you-are-doing-labs)
- [After Doing Labs](#after-doing-labs)

[Techniques and Tricks for OSCP](#techniques-and-tricks-for-oscp)
- [Enumeration Tricks](#enumeration-tricks)
- [Exploiting](#exploiting)
- [Reverse Shells](#reverse-shells)
- [File Upload Vulnerabilities](#file-upload-vulnerabilities)
- [More Web Service Tips](#more-web-service-tips)
- [SSH Enumeration](#ssh-enumeration)
- [Linux Privilege Escalation](#linux-privilege-escalation)
- [Windows Privilege Escalation](#windows-privilege-escalation)
- [AD Tips](#ad-tips)

[Conclusion](#conclusion)

---
# Resources
**Before enrolling in PEN-200**: if you are deciding to get the OSCP, and you don't have a time constrain and wants to try to guarantee your pass on the first attempt, I recommend you to start learning from [Hack The Box Academy](https://academy.hackthebox.com/) - [Penetration Tester Path](https://academy.hackthebox.com/path/preview/penetration-tester) for the HTB Certified Penetration Testing Specialist (CPTS) certificate. A lot of people consider the materials taught in CPTS much better than the material taught in OSCP, yet it is cheaper. I believe completing the path could range from 3 to 6 months, but you may also skip some modules that are irrelevant to the OSCP exam if you like. 
*Note that the OSCP exam is harder than CPTS in a time constrained way (you only have 24 hours), but the CPTS exam is harder in terms of skills. Therefore you do **NOT** need to take the CPTS exam if your goal is to pass the OSCP exam ASAP.*

![](https://academy.hackthebox.com/storage/exam_og_images/0nGJoe89AqXCilr858s1D4gB7AiBE0F0vUZczUQs.png)

Now, **if you are already enrolled in PEN-200**: and you want more hands-on training before the exam, you may reference to [LainKusanagi's OSCP Like List](https://docs.google.com/spreadsheets/d/18weuz_Eeynr6sXFQ87Cd5F0slOj9Z6rt/) or [TJNull's List](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview): *I like the LainKusanagi's list more* because I feel it's more updated, but you may choose either:
### Proving Grounds Practice
These are the **must do** labs from both lists, they are the most similar to the exam; you may do them first or later near the exam, but the work flow of the labs are **intended to feel the same as OffSec's CTF style**. You will get a decent feeling of what you may encounter during the OSCP exam, but the exploits/techniques used in the exam may require you to become **more creative**
### Hacker Blueprint AD Chains
**The absolute golden beast for practicing AD Set** in OSCP. Especially after the release of AD Chains 7 and 8, **they are very similar and good practice for the OSCP AD Set**. You can learn very important  techniques which is **crucial to the AD Set and Windows stand-alones in OSCP.**
*Note: AD Chains 1-6 from Hacker Blueprint requires VirtualBox for best compatibility*
### Virtual Hacking Labs
I have a lot of love and hate for this platform, it doesn't have the best UI and support for sure, but it taught me the most about how to think when attacking a Linux machine; also how to think about web services in a more clear view. **I would say only do this platform if you think you are weak in Linux.** On the other hand, I do think I rooted the linux machine in my OSCP exam very fast because I had a lot of practice from this platform.
### Hack Smarter
This platform is owned by Tyler Ramsbey I believe, I came across his youtube videos before and I beleive the AD practices are very high quality, and so are the linux machines. Although the labs are very high quality, but **the Linux labs are less relevant to OffSec style in my opinion**, it does have more challenge to "try harder" but I think in a different approach than OffSec.
### Hack The Box
Although Hack The Box is my favorite platform when it comes to cyber security, I do think **it is not the best practice for OSCP.** Doesn't mean it's easier, like Hack Smater, Hack The Box approach is also quite different from OffSec even more than Hack Smarter. But like LainKusanagi says, it teaches some important concepts: so say either watch [IppSec](https://www.youtube.com/@ippsec)  or read through [0xdf writeups](https://0xdf.gitlab.io/)
![](https://cdn.tech.eu/uploads/2019/04/Hack-The-Box-logo.png)
### Proving Grounds Play
I would say you can do these labs from the list when you are done with everything, these labs do teach some pretty good methodologies and knowledge too, but **Proving Grounds Practice has better quality in general** in my opinion.
### PEN-200 Challenge Labs
**Extremely good practice for the AD set in OSCP,** I think some are little out of scope from the OSCP exam itself, but definitely **do OSCP A, B and C as mock exams**. The other ones I did and think were good practice as well are: **Secura, Zeus, Poseidon and Laser**. You can also do the other challenge labs since they are also good, but I did not feel the need to.

---
# Preparation Tips
### Before Doing Labs
- **Set up a kali VM**: I recommend VMWare; it's smoother from my experience
- **Decide an application for note taking**, the most popular ones are **OneNote**, **CherryTree**, etc. I personally use **Obsidian** since the UI are beautiful to me; another main reason is I can copy and paste my commands easily with `code blocks` in Obsidian
- **Make sure you go over the PEN-200 course material** and at least try to understand and make sense of everything. Meanwhile, try to do the labs in most of the chapters
- Finished the "boring" course materials? Now **Plan for your practice paths:** maybe you want to do all of the Proving Grounds Practice first like I did? Or that you want to do HTB boxes first? That's fine, having a plan is much better than not having a plan.
### While You are Doing Labs
- **Create a template for labs** - then take notes such as `nmap` and enumeration results, exploits, etc. I believe taking notes will help you visualize your attack paths in the long run, but you can start with taking very simple notes to slowly taking more complicated notes later
- **Do NOT hesistate to ask for hints on places like Discord or look up walkthroughs** after you tried your methodology/techniques at least twice. When you get stuck, it doesn't mean you are stupid, it just means you need to learn new methodology or get creative (**try harder**).
- **After every single lab, take time to *digest* the new methodology you learned**. Make a section in your template called "Lessons Learned" if you want. Write down things you didn't do well, you did well on, and what you learned in which you will be applying them in future labs
- After every tool/script you learned how to use, **save the new tool/scripts to your notes or your kali VM**. I have a `/opt/all_tools` directory that I use with `evil-winrm` or hosts a python http server on to transfer scripts
- (*Optional*) I made a exploits bank which I noted down all of the exploits I ever encountered and the working exploits that I've used. Just incase I see similar version and services during the exam, or even in future labs I know what exploits to use right away.
![](https://obsidian.md/images/banner.png)
### After Doing Labs
This is actually very important: before the exam, **take a break from cyber security, do NOT even look a single word about cyber security**. For how long is up to you, but I recommend not doing anything for a full 1 to 3 days. Do anything you like such as playing games, play sports, whatever you want and enjoy before the exam. **Do NOT go in to exam with cyber security burnout** like I did.

---
# Techniques and Tricks for OSCP
I have to say these are not the most organized notes; but I think very detailed. For those of you that are reading this, you can use this as a reference to when you are practicing and/or add to your methodology.
### Enumeration Tricks
I won't list how to enumerate every services from my mythodology which is not complete, I google all the time and I believe knowing how to google is very important to being a good pentester. 

- Reference to the 3 famous sites when enumerating for a service or even PrivEsc. When googling you can type for example: `wordpress enuemration` and look for results from:
	1. [HackTricks](https://hacktricks.wiki/en/index.html): the swiss army knife wiki for pentesting in my opinion
	2. [Hackviser](https://hackviser.com/tactics/pentesting): I find hackviser to have the best quality of enumeration tricks so far
	3. [Hacking Articles](https://www.hackingarticles.in/): blog website by cybersecurity professional `Raj Chandel`, can include very handy tricks such as PrivEsc on Linux using the `docker` user group
- **Try default and simple credentials for, every, single, service, open**. If you found any usernames and passwords, also try to spray them on every single service open too. 
	- simple credentials include: `admin:admin`, `root:password`, `root:root`, etc.
- Also, if you only found usernames, **spray their usernames as passwords** on every, single, service, open, too. I cannot emphasize this more since I always end up doing this last when this is the thing keeping out from rooting the lab.
- If you got into a database service like `mysql` and can't crack the password for the user; you can try looking for the default password hash and update the field so we can "change" the password
### Exploiting
- Learn to look for web service version number or exploits with the **web service corresponding tools**: the most classical example being `wpscan` for `wordpress`, or `joomscan` for `joomla`
- Once you have found the web service name and the corresponding version number, there are typically 2 places I look for exploits:
	1. `exploit-db` or using `searchsploit` command, or:
	2. Googling `[service name] [version number] exploit site:github.com`
	    This will limit the search results to be from `github` with google dorking; If this PoC doesn't work, try a couple more PoC from `github`/`exploit-db` if you are certain
  Note that: sometimes the exploit could still work for **service numbers that are not matching**
- Always **read exploit description** first for instructions to using the exploit PoC, then read through the exploit and have a general understanding of how it works and **modify accordingly**.
- If the exploit is based on web service and is not working, **make sure the path specified in the exploit is correct**. Sometimes exploit may be using `/wordpress` when lab is using `/wp`
- If you are dealing with python exploits, you might need to **specify the python version** to run the exploit properly: such as `python2 [exploit.py]` or `python2.7 [exploit.py]` 
- Again, if you are dealing with python exploits, you might have to learn how to use **python virtual environment** to install/implement missing modules
### Reverse Shells
The absolute precious resource for reverse shells: https://www.revshells.com/ by [0day](https://ryanmontgomery.me/)

- If the goold old `bash -i` doesn't work, **you may have to wrap it with `bash -c 'bash -i'`**
  (research on why this works as a homework!)
- Sometimes the `echo [base64 reverse shell] | base64 -d | bash` could work, but I typically get away with `bash -c 'bash -i'` reverseshell in OSCP labs
- **Try the `busybox` reverse shell** as sometimes the lab machine doesn't have `nc` or disabled `nc`
- Always catch reverse shells from Linux with `nc -lvnp [port]` **upgrade the initial shell**
  *Note: you cannot fully upgrade a linux shell with `rlwrap nc` from my experience*
- When dealing with **Windows** machines: set up listener using `rlwrap nc -lvnp [port]` as this allows for a more stabilized shell. In addition, use the `PowerShell #3 (Base64)` as payload.
- **Try common ports** to catch reverse shells: such as `80` or try some of the ports target has open
- Try a couple more different reverse shell commands from https://www.revshells.com/ if you are **sure that your exploit is the correct exploit**
- **Always encode your reverse shell command with URL encoding** if sending it with web request
![](https://media.tcm-sec.com/uploads/2023/01/reverse-shell-1024x333.png)
### File Upload Vulnerabilities
- If you uploaded a reverse shell and can't seem to find the directory it is in: leverage `gobuster` reults or view source with `Ctrl + U` to find similar links for clues.
- **Bypass php extension checks** by either using `.php7`, `.phtml`, etc.
- **Bypass simple file extension checks** by intercepting and modifying with `Burp Suite`
- Sometimes we can also **make use of file sharing services** to upload reverse shells if it has writing permissions and can access the web directory. For example we can use SMB, FTP, etc. to upload the shell and trigger it.
- Triggering uploaded reverse shell: usually we can trigger it directly through web requests, but sometimes we can also **leverage Local File Inclusion (LFI) to trigger reverse shell**
### More Web Service Tips
- Look at the source code of pages that doesn't look like popular web services (such as Wordpress, Joomla, Jenkins, etc.) and look for hints for credentials or any clues, think of it as you are playing a detective game. If there are domain name information such as `test.htb`, add that in your `/etc/hosts` file and enumerate for subdomains with `ffuf` or `gobuster vhosts`
- Leverage `nikto` if needed, I personally didn't find it too interesting. However, it can yield results for hints to initial access sometimes.
- When enumerating a popular/well known web service, we can enumerate with their corresponding wordlists from seclists. For example we can search apache wordlists by 
  ```
  ls -R /usr/share/wordlists/seclists/Discovery/Web-Content | grep -i "apache"
  ```
  and we can find the corresponding wordlist by
  ```
  locate Apache.txt
  ```
  then we can use that wordlists with `gobuster` or `ffuf`
- If the target webservice has `.git`, we can use [`git-dumper`](https://github.com/arthaud/git-dumper) to dump git repositories which can leak crucial information
- When testing for LFI vulnerabilities, try doing a lot of `../` in requests with parameters such as `?view=` or `?page=` 
- When enumerating for Windows IIS servers, try enumerating with `.aspx` and `.asp` if you can't find anything with `.php` extensions in `gobuster`
![](https://i0.wp.com/bydik.com/wp-content/uploads/2016/01/WordPress-logo-black-765-e1664447823401.jpg?fit=1000%2C550&ssl=1)
### SSH Enumeration
This may become repetitive but it's very important: **try `username:username` of any usernames you found from other services** to see if you can log in; **try spraying and not limited to SSH**.
### Linux Privilege Escalation
I organized for a few major types of privilege escalation types, and I think if they are not in them it's usually that I have to **get creative**. Linux PrivEsc is something that can be both fun and painful. I will exclude out some of the "common sense" ones and add ones I think I missed out a lot on:

1. Look in web service config files for passwords to access database (typically `mysql`) or spraying that password against users from `/etc/passwd`: such as `wp-config.php` for `wordpress`. 
2. If we found passwords, we can also try using them in web services or other services to gain further access to potentially another user.
3. Enumerating strange file in `/opt`, `/var`, the mails if there is a mail service, `/home/[user]` of other users. Also, the `.bash_history`, `.bashrc`, etc. of currently logged in user.
4. Enumerate our current user, we may have special user groups that can leverage special privilege escalation methodologies. Or that we belong to unusual group that can read/write on files that can only be read/write to that group. And also look in `env`, `history`, etc.
5. Look in `/etc/passwd`; and **try username as password for users**, again.
6. If we get to laterally move to another user, we may need to re-enumerate everything again (including `sudo` privileges, `SUID` bit set programs, etc.)

If we still cannot obtain root after double enumeration, I would try to leverage something **more creative.** This could include abuse modifying `PATH` variable with cronjobs, kernel exploits (not really more creative, but less often in current OSCP exams), etc.
### Windows Privilege Escalation
In my opinion, pretty straight forward in OSCP, personally the hard part is credential hunting.

- `whoami /all`: you should know how to abuse tokens such as `SeImpersonate`, `SeBackup`, etc.
- `whoami /groups`: abuse special groups such as `Service Operator`, `LAPS_Reader`, etc.
- Enumerate for `AlwaysInstallElevated`: create `.msi` `msfvenom` payload 
- Insecure service permissions: learn `PowerUp.ps1` and `PrivescCheck.ps1`
	- When making a `msfvenom` payload, we generate `exe-service` instead of `exe`. We also might need to change to staged payload and catch with `msfconsole` if unstable.
	- Remeber: `SeShutdown` token doesn't need to be enabled to restart Windows
	- If we can't restart with `CMD` or `PowerShell` then we can try using `RDP` to log in and restart manually with the Windows button.
- Search for credentials: in `PowerShell`/`CMD` history, etc.
  *Note: AD Chains 7 from Hacker Blueprint teaches a very detailed methdology to search for credentials in Windows* 
![](https://yl-labs.github.io/assets/imgs/netexec-mastery/cover.png)
### AD Tips:
- **Practice `netexec`!** The absolute most useful tool in AD. It can basically do everything
	- Learn to make a user list with the output of `netexec smb` (also try `--rid-brute`)
	- Note down all of the passwords you collect, and spray them on **all users**.
	- Spray the users list as passwords against all users
	- Use `netexec` to confirm if you can log in with `winrm` or `rdp` (shows golden `(Pwn3d!)`)
- **Run bloodhound** as soon as you finish with enumerating services, this can show a more detailed and clear attack path
- When leveraging bloodhound: enumerate domain admins and **all of the "shortest paths"**
	- Bloodhound will give you hints on abusing misconfigured ACEs, but they are not copy and paste and sometimes you might need to do some debugging.
- Learn to enumerate `MSSQL`: impersonate other users, code execution, NTLM capture, etc. 
  [hackviser](https://hackviser.com/tactics/pentesting/services/mssql) has a good list of enumerations that **is important in OSCP AD Set**
- Learn and practice how to search for credentials; like I stated in Windows PrivEsc section above, the **AD Chains from Hacker Blueprint** can be great to practice (especially Chain 7 and 8)
- **Pivoting:** not actually that complicated in OSCP, `ligol-ng` is basically all you need. However, I prefer pivoting internally to a service of a machine with `chisel`
- **NTLM capturing** is possible in a lot of scenerios when you can trick the target into accessing your kali machine; if cracking hashes seems impossible, try relaying it.
- Enumerate the `SYSVOL` for `groups.xml`, may contain `cpassword`; decrypt with `gpp-decrypt`
- After you got privileged shell on Windows, add a domain user as local admin and dump `LSASS`, `LSA`, `SAM`, etc. with `netexec` 
---
# Conclusion
Thank you all for coming this far, I might update the tips more along my cyber security journey but I truly believe that **practice makes perfect.** In addition, if you have tried my tips and tried everything from your methodology and can't find the way. Maybe it's time to calm down and think for a moment, and try **getting creative**. I dislike the wording of OffSec's way of saying "try harder", because that feels like  we are not trying hard and that's simply not true. Sometimes we are just less experienced or not getting creative enough in my opinion.

On my 100 points OSCP attempt, I got the last 20 points with something that I barely touched on before, but luck was on my side and something clicked in my head into pwn'ing the whole OSCP exam in just under 7 hours.

Last but not least, I wish all of you the best of luck, not just in you OSCP exam that's coming up in 2 days, or maybe that you are doing the OSCP exam as of the very moment you are reading this line, but also your career. And this blog will conclude the end of my OSCP journey!

Feel free to reach out to my discord `hiroto` or on LinkedIn if you have any questions.

ON to the next.