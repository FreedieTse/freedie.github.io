+++
date = '2026-06-03T10:50:00-04:00'
title = 'Bashed'
tags = ["htb", "easy","sudo","cronjobs"]
+++

``` IP
10.129.186.17
```
# OS:  Ubuntu
# Credentials:

| Username | Password |
| -------- | -------- |


---
# `nmap` results:

```
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 6AA5034A553DFA77C3B2C7B4C26CF870
|_http-title: Arrexel's Development Site   
| http-methods:                      
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
Device type: general purpose         
Running: Linux 3.X|4.X 
OS CPE: cpe:/o:linux:linux_kernel:3 cpe
```

---
# Attack + Enum Vectors: (Ports, etc.)
- TCP 80: HTTP

---
# Web Service Enum:
## `Gobuster` / `wfuzz`
after enuming with gobuster: we found a `/dev` directory with web shell
```
http://10.129.186.17/dev/phpbash.min.php
```
and
```
http://10.129.186.17/dev/phpbash.php
```


---
# Initial Foothold
```
http://10.129.186.17/dev/phpbash.php
```
then uploaded a php reverse shell on `/uploads` directory
then we visited that on browser and caught a shell
now we try to privilege escalate

---
# Priv Esc
First we transfer pspy64 to target: we found a cron job running the python script under scripts file

we first lateral move to scriptmanager using 
```
sudo -u scriptmanager bash -i
```
then we make a reverse shell python script and place it in scripts directory. which will be executed by root every minute; then set up listener
```
sudo nc -lvnp 80
```
we got root

Therefore, pwn'd