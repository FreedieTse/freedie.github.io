+++
date = '2026-06-11T01:59:34-04:00'
title = 'Busqueda'
tags = ["htb", "easy","repeat"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/a6942ab57b6a79f71240420442027334.png)

https://www.hackthebox.com/machines/Busqueda
## OS: Linux
``` IP
10.129.187.196
```
## Credentials:

| Username        | Password              |
| --------------- | --------------------- |
| `cody`          | `jh1usoih2bkjaspwe92` |
| `svc`           | `jh1usoih2bkjaspwe92` |
| `administrator` | `yuiu1hoiu4i5ho1uh`   |

---
## `nmap` results:
```
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:                       
|   256 4f:e3:a6:67:a2:27:f9:11:8d:c3:0e:d7:73:a0:2c:28 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIzAFurw3qLK4OEzrjFarOhWslRrQ3K/MDVL2opfXQLI+zYXSwqofxsf8v2MEZuIGj6540YrzldnPf8CTFSW2rk=      
|   256 81:6e:78:76:6b:8a:ea:7d:1b:ab:d4:36:b7:f8:ec:c4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPTtbUicaITwpKjAQWp8Dkq1glFodwroxhLwJo6hRBUK  
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://searcher.htb/
```

---
## Web Service Enum:
redirected to `http://searcher.htb/search`

subdomain enumerating and gobustering we got:
```
http://gitea.searcher.htb/administrator
```
```
http://gitea.searcher.htb/user/login
```

---
## Attack + Enum Vectors: (Ports, etc.)
- TCP 80: HTTP Apache httpd 2.4.52
- TCP 22: SSH OpenSSH 8.9p1

---
## Initial Access
Searching google for exploits lands us this:
https://github.com/twisted007/Searchor_2.4.0_RCE_Python/blob/main/searchor-2_4_0_RCE.py

which we use to get initial access

---
## Priv Esc
```
sudo -l
```
reveals:
```
sudo /usr/bin/python3 /opt/scripts/system-checkup.py
```
After getting some hints by watching ippsec: the above pass seem to work like a docker command?
therefore after seeing him analyzes docker content with json i start trying by myself

Now after analyzing with `--help` we see there are three different scripts we can run:
docker-ps: shows the information about current running dockers, docker-inspect: inspects data of the current dockers, and full-checkup: a `.sh` scripts

and looking in `/opt/scripts` we can run the scripts but not edit them. So after researching online about the scripts: seems like they are similar to the commands:
```
https://docs.docker.com/reference/cli/docker/inspect/
```
and using docker-ps we note down the id and name of the dockers
```
https://docs.docker.com/engine/cli/formatting/https://www.hackthebox.com/machines/Box
```
then we try running inspect: one of the options from above link has some ways that can view data: we tried the `{{json .Mounts}}` first but seemed useless info: at the bottom of the page tho says
`'{{json .}}'` shows all content as json

and also online says that we can use pump output into `jq` for prettier output

then we `grep -i pass` and found a password; and we tried it on `gitea.searcher.htb:` we get to log on as admin

then logging on we cannot edit the script files: I tried to edit the configurations on the website but doesn't seem to work; watching ippsec: the `./full-cleanup.sh` or which means it is executing it at the directory when we use sudo and the command; therefore we created a bash script for reverse shell at `tmp` or some directory we can edit, however:

It didn't work for some reason; I realized I needed to add `chmod +x` to the script too doesn't matter it is executed through sudo, also i need to add `#!/bin/bash` at begging of the script then we get root
## Conclusion & Remediation
This lab was also done before I started blogging my write-up. Honestly, this is pretty messy writing. However, I did learn a lot including to use `jq` to make the output of `.json` files  prettier and nicer to read, improving my researching skills on things I haven't encountered before to adapt accordingly.

To remediate for such attacks: system administrators need not just to hide their web application on a subdomain, but also update them to patch from critical exploits like the ones appeared. In addition, the misconfigured `sudo` scripts needs to be removed to follow the principle of least privileges.