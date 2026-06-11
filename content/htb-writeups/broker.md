+++
date = '2026-06-01T23:49:11-04:00'
title = 'Broker'
tags = ["htb", "easy", "sudo","nginx","activemq"]
+++

![](https://htb-mp-prod-public-storage.s3.eu-central-1.amazonaws.com/avatars/a725533911ba94a880899fbf900d988c.png)

https://www.hackthebox.com/machines/Broker
## OS: Linux
``` IP
10.129.230.87
```
## Credentials:

| Username | Password |
| -------- | -------- |

---
## `nmap` results:

```
PORT      STATE SERVICE    REASON         VERSION
22/tcp    open  ssh        syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp    open  http       syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-title: Error 401 Unauthorized
|_http-server-header: nginx/1.18.0 (Ubuntu)
1883/tcp  open  mqtt       syn-ack ttl 63
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     ActiveMQ/Advisory/MasterBroker: 
|_    ActiveMQ/Advisory/Consumer/Topic/#: 
5672/tcp  open  amqp?      syn-ack ttl 63
|_amqp-info: ERROR: AQMP:handshake expected header (1) frame, but was 65
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GetRequest, HTTPOptions, RPCCheck, RTSPRequest, SSLSessionReq, TerminalServerCookie: 
|     AMQP
|     AMQP
|     amqp:decode-error
|_    7Connection from client using unsupported AMQP attempted
8161/tcp  open  http       syn-ack ttl 63 Jetty 9.4.39.v20210325
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-title: Error 401 Unauthorized
|_http-server-header: Jetty(9.4.39.v20210325)
40335/tcp open  tcpwrapped syn-ack ttl 63
61613/tcp open  stomp      syn-ack ttl 63 Apache ActiveMQ
| fingerprint-strings: 
|   HELP4STOMP: 
|     ERROR
|     content-type:text/plain
|     message:Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolException: Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolConverter.onStompCommand(ProtocolConverter.java:258)
|     org.apache.activemq.transport.stomp.StompTransportFilter.onCommand(StompTransportFilter.java:85)
|     org.apache.activemq.transport.TransportSupport.doConsume(TransportSupport.java:83)
|     org.apache.activemq.transport.tcp.TcpTransport.doRun(TcpTransport.java:233)
|     org.apache.activemq.transport.tcp.TcpTransport.run(TcpTransport.java:215)
|_    java.lang.Thread.run(Thread.java:750)
61614/tcp open  http       syn-ack ttl 63 Jetty 9.4.39.v20210325
|_http-server-header: Jetty(9.4.39.v20210325)
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|   Supported Methods: GET HEAD TRACE OPTIONS
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title.
61616/tcp open  apachemq   syn-ack ttl 63 ActiveMQ OpenWire transport 5.15.15
```
---
## Attack + Enum Vectors: (Ports, etc.)
- TCP 8161: HTTP Jetty 9.4.39
- TCP 1883: MQTT?
- TCP 5672: AMQP?
- TCP 61613: stomp Apache ActiveMQ?
- TCP 61614: HTTP Jetty 9.4.39
- TCP 80: HTTP nginx 1.18.0
- TCP 40335
- TCP 22: SSH OpenSSH 8.9p1

---
## Initial Foothold
searching up the exploit for the version of activemq we found an exploit:
```
https://github.com/rootsecdev/CVE-2023-46604
```
it can execute and force the server to make a request to us to execute a code for reverse shell in the directory of
```
/opt/apache-activemq-5.15.15/bin
```

---
## Priv Esc
`sudo -l `has something with nginx without need of password: but did not seem promising after trying https://gtfobins.org/gtfobins/nginx/

After watching ippsec he seemed to used the same way i tried. I retried: this time doesnt work too. Now i studied how this works again and appearantly it copies the public key from current `.ssh` of current direcotry to root `.ssh` directory; therefore i changed directory to my home then re-generated a key and followed https://gtfobins.org/gtfobins/nginx/ and finally: we uploaded the `id_rsa`, using the key with `ssh` and we got root.

Therefore pwn'd.

---
## Conclusion & Remediation
I did this box before I started writing my write ups on my blog so it is not the best formatted. However, I learned that trying to understand how everything works is important since if it doesn't work initially: doesn't mean it doesn't work, could mean we're not using it properly or lacking some knowledge

To remediate for similar attacks from this lab: system administrators needs to make sure to not use vulnerable web message brokers like the `activemq` in this lab, and update them. In addition, misconfigured `sudo` privileges needs to be removed to remediate for privilege escalation attacks and to follow the principle of least privileges.