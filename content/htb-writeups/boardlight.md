+++
date = '2026-06-01T23:49:11-04:00'
title = 'BoardLight'
tags = ["htb", "SUID exploit", "easy","password guess"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/7768afed979c9abe917b0c20df49ceb8.png)

https://www.hackthebox.com/machines/BoardLight
## OS: Linux
``` IP
10.129.186.128
```
## Credentials:

| Username        | Password            |
| --------------- | ------------------- |
| `dolibarrowner` | `serverfun2$2023!!` |

---
## `nmap` results:
```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA) 
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDH0dV4gtJNo8ixEEBDxhUId6Pc/8iNLX16+zpUCIgmxxl5TivDMLg2JvXorp4F2r8ci44CESUlnMHRSYNtlLttiIZHpTML7ktFHbNexvOAJqE1lIlQlGjWBU1hWq6Y6n
1tuUANOd5U+Yc0/h53gKu5nXTQTy1c9CLbQfaYvFjnzrR3NQ6Hw7ih5u3mEjJngP+Sq+dpzUcnFe1BekvBPrxdAJwN6w+MSpGFyQSAkUthrOE4JRnpa6jSsTjXODDjioNkp2NLkKa73Yc2DHk3evNUXfa+P8oWFBk8ZXSHFy
eOoNkcqkPCrkevB71NdFtn3Fd/Ar07co0ygw90Vb2q34cu1Jo/1oPV1UFsvcwaKJuxBKozH+VA0F9hyriPKjsvTRCbkFjweLxCib5phagHu6K5KEYC+VmWbCUnWyvYZauJ1/t5xQqqi9UWssRjbE1mI0Krq2Zb97qnONhzcc
lAPVpvEVdCCcl0rYZjQt6VI1PzHha56JepZCFCNvX3FVxYzEk=
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBK7G5PgPkbp1awVqM5uOpMJ/xVrNirmwIT21bMG/+jihUY8rOXxSbidRfC9KgvSDC4flMsPZUrWziSuBDJAra5g=
|   256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)                   
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILHj/lr3X40pR3k9+uYJk4oSjdULCK0DlOxbiL66ZRWg  
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
```

---
## Attack + Enum Vectors: (Ports, etc.)
- TCP 80: HTTP Apache httpd 2.4.41
- TCP 22: SSH OpenSSH 8.2p1

---
## Web Service Enum:
### `Gobuster` / `fuff`
### Subdomain:
Using `ffuf` subdomain brute forcing: we found `crm.board.htb`

---
## Initial Foothold
`crm.board.htb` shows `dolibarr 17.0`; trying default credential `admin:admin` we logged in;
but looking around we can't seem to find anything to upload or malicious

Searching on google we found the exploit with reverse shell:
https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253/blob/main/exploit.py

Initiate a reverse shell and caught it on our listener and we got in as www-data

---
## Priv Esc
under `html/crm.board.htb/htdocs/conf/conf.php.old`
```
// $dolibarr_main_db_pass='myadminpass';
// $dolibarr_main_db_pass='myuserpassword';
$dolibarr_main_db_pass='';
// values using a ",". In this case, Dolibarr will check login/pass for each value in
// $dolibarr_main_authentication='dolibarr';       // Use the password defined into application on user file (default).
// $dolibarr_main_authentication='ldap';
```

and under `html/crm.board.htb/htdocs/conf/conf.php`
```
$dolibarr_main_db_user='dolibarrowner';
$dolibarr_main_db_pass='serverfun2$2023!!';
```

`mysql -u dolibarrowner -p'serverfun2$2023!!' -h localhost`

logging into mysql with the above credentials we get two things:
```
$2y$10$VevoimSke5Cd1/nX1Ql9Su6RstkTRe7UX1Or.cm8bZo56NjCMJzCm
$2y$10$gIEKOl7VZnr5KLbBDzGbL.YuJxwz5Sdl5ji3SEuiUSlULgAhhjH96
```
first being dolibarr; second admin

other suspicious files:
`html/crm.board.htb/htdocs/asset/admin/setup.php`
`html/crm.board.htb/htdocs/adherents/note.php`
`html/crm.board.htb/htdocs/adherents/htpasswd.php`

After trying a bunch of things and watching ippsec: i realized there was one thing i missed: try the database password on the `larissa` acount i found that hash shell

So we do `su larissa` and enter password `serverfun2$2023!!`

Then utilizing the exploit of enlightment on SUID: we got to escalate our privileges to root;
https://github.com/d3ndr1t30x/CVE-2022-37706/blob/main/exploit.sh

---
## Conclusion & Remediation
I did this box before I started writing my write ups on my blog so it is not the best formatted. I did reviewed this write up right after I finished OpenAdmin, and wow I really need to add spraying database password on all users to my methodology now because I keep forgetting that. I also leanred when we want to try to escalate privileges and see SUID bit programs that are odd/weird, we should note them down to search online for privilege escalation possible exploits.

To remiate from similar attacks in this lab: system administrators should be updating their outdated web application, and avoid using default/simple credentials such as `admin:admin`. In addition, passwords used for databases should not be re-used for users passwords. SUID bit set programs should also be checked on a daily basis if they are vulnerable to local privilege escalation vulnerabilities. 