+++
date = '2026-06-01T23:49:11-04:00'
title = 'Codify'
tags = ["htb", "easy","authentication bypass","pspy"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/57b977ea744af01a5454c8643a850e59.png)

https://www.hackthebox.com/machines/Codify
## OS: Linux
``` IP
10.129.188.8
```
## Credentials:

| Username | Password     |
| -------- | ------------ |
| `joshua` | `spongebob1` |

---
## `nmap` results:
```
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:         
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN+/g3FqMmVlkT3XCSMH/JtvGJDW3+PBxqJ+pURQey6GMjs7abbrEOCcVugczanWj1WNU5jsaYzlkCEZHlsHLvk=
|   256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIm6HJTYy2teiiP6uZoSCHhsWHN+z3SVL/21fy6cZWZi
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.52
| http-methods:        
|_  Supported Methods: HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://codify.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
3000/tcp open  http    syn-ack ttl 63 Node.js Express framework
|_http-title: Codify
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```

---
## Web Service Enum:
### `Gobuster` / `fuff`:
#### Directories
`/about`
`/editor`

---
## Initial Foothold
`/about` shows that they are using `3.9.16` `vm2` which has a vulnerability:

using https://www.exploit-db.com/exploits/51898 and executing reverse shell with `busybox nc` we obtained a reverse shell

---
## Priv Esc
`/opt/scripts/mysql-backup.sh`:
```
#!/bin/bash
DB_USER="root"
DB_PASS=$(/usr/bin/cat /root/.creds)
BACKUP_DIR="/var/backups/mysql"

read -s -p "Enter MySQL password for $DB_USER: " USER_PASS
/usr/bin/echo

if [[ $DB_PASS == $USER_PASS ]]; then
        /usr/bin/echo "Password confirmed!"
else
        /usr/bin/echo "Password confirmation failed!"
        exit 1
fi

/usr/bin/mkdir -p "$BACKUP_DIR"

databases=$(/usr/bin/mysql -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" -e "SHOW DATABASES;" | /usr/bin/grep -Ev "(Database|information_schema|performance_schema)")

for db in $databases; do
    /usr/bin/echo "Backing up database: $db"
    /usr/bin/mysqldump --force -u "$DB_USER" -h 0.0.0.0 -P 3306 -p"$DB_PASS" "$db" | /usr/bin/gzip > "$BACKUP_DIR/$db.sql.gz"
done

/usr/bin/echo "All databases backed up successfully!"
/usr/bin/echo "Changing the permissions"
/usr/bin/chown root:sys-adm "$BACKUP_DIR"
/usr/bin/chmod 774 -R "$BACKUP_DIR"
/usr/bin/echo 'Done!'
```

Aoing into `/var/www/html/` we see a web directory that is not listed on the website; listing shows that there's a `.db` file. After dumping it with `sqlite3 [.db] .dump` we get a hash

After cracking it with hashcat `-m 3200`: we got the pass of `spongebob1`. Then reusing the pass we logged into joshua with that pass

`sudo -l` shows that we can execute a backup bash script: we have to supply password though, and I tried spraying password and other ways, doesn't seem to work.

[ippsec](https://www.youtube.com/watch?v=wH1Lp-sEVv4) showed show we can bypass authentication with a wildcard: since the script is using bash `if [[ == [input]]]` to compare password. We can abuse it with authentication bypass by supplying `*` wildcard. Finally we get to run script as `sudo`
But we can't see the backup directory: what to do now? [ippsec](https://www.youtube.com/watch?v=wH1Lp-sEVv4) showed that since the script is supplying a password that was stored from reading root's `.cred` file; we can maybe spoof it using `pspy`

Let's run `pspy`, then `sudoing` bypass authentication: we get a password!
Using that password for  root: we get root shell

Therefore, pwn'd.

---
## Conclusion & Remediation
I did this box before I started writing my write ups on my blog so it is not the best formatted. The privilege escalation in this box was very interesting; utilizing `*` to bypass authentication is a neat and interesting trick that might help me down the line as a penetration tester.

To remediate for such similar attacks in this lab: system administrators needs to ensure any services running on their web server are up to date and not vulnerable to any public vulnerabilities. In addition, misconfigured `sudo` scripts needs to be removed to follow the principle of least privileges. If string comparison is used toi confirm supplied string is equal to a password, the input value compared should be surrounded by `""` double quotes to prevent similar authentication bypass in this lab.