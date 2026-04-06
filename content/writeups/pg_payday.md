---
title: "PG Practice - Payday"
date: 2026-04-04
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
Nmap scan report for 192.168.191.39
Host is up (0.033s latency).
Not shown: 65527 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
80/tcp  open  http        Apache httpd 2.2.4 (PHP/5.2.3-1ubuntu6)
110/tcp open  pop3        Dovecot pop3d
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
143/tcp open  imap        Dovecot imapd
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
993/tcp open  ssl/imaps?
995/tcp open  ssl/pop3s?
Service Info: Host: 127.0.0.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Local

 To start this box, I manually checked the Webserver running on port **80** on the target machine. There was a **CS-Cart** instance running and i noticed the login-section. I tried some default credentials and _admin:admin_ worked. 
 
 Next, I was able to find the admin interface `/admin.php` by fuzzing. In the admin interface i discovered the used version of **CS-Cart** in the upgrade center under `admin.php?target=upgrade_center`: 

 ![](/images/writeup_screens/pg_payday/pg_payday1.png)

 ```
CS-Cart Version: 1.3.3
 ```

 I searched the web for vulnerabilities affecting this version and found an authenticated RCE with a [public exploit](https://www.exploit-db.com/exploits/48891). I followed the exploit and uploaded a **PHP web shell** as _.phtml_ in the **template editor**:

 ![](/images/writeup_screens/pg_payday/pg_payday2.png)

 The uploaded shell is accessible via the `/skins/` directory and is functional as can be seen in the following screenshot:

 ![](/images/writeup_screens/pg_payday/pg_payday3.png)

 Subsequently, i started a **netcat** listener on my attacker machine and uploaded a **PHP reverse shell** (e.g. from PentestMonkey) in the same manner as above. After visiting it in the `skins` directory i received a reverse shell in my listener as user **www-data**. 

 We can now simply read _local.txt_ from the home directory of **patrick**.

 > The **CS-Cart** instance is also vulnerable to a **LFI** which allows us to read _/etc/passwd_. Although a shell on the machine is obviously better from an attacker perspective, reading _/etc/passwd_ and discovering that there is a user **patrick** on the machine, would be sufficient in this case. This will make sense after reading the section for the **Proof** flag.

 ![](/images/writeup_screens/pg_payday/pg_payday4.png)

---

## Proof

The first thing we should do is to get a TTY shell in our obtained reverse shell with `$ python -c 'import pty; pty.spawn("/bin/bash")'`. 

There is really not much to say about this part of the machine as we can just get a shell as user **patrick** by using the password _patrick_. `$ sudo -l` reveals that this user has basically root privileges and we can read the flag at _/root/proof.txt_:

![](/images/writeup_screens/pg_payday/pg_payday5.png)

> While this is very simple, it is in my opinion also very unusual to have an attack path like this. I got caught up in some rabbit holes and wasted a lot of time before trying username = password. Another option would be to brute force the **SSH-login** with **hydra** but this is also a pretty uncommon thing in challenges like this. 