---
title: "HTB - Cap"
date: 2026-02-28
draft: false
tags: ["HTB", "Linux", "Write-Up"]
---

## User Flag

Nmap Scan shows open ports: 21, 22 and 80

On port 80 there is a web application in which we are logged in by default as the user **Nathan**. In the application we can check out different network stuff. When we click the burger menu and select **Security Snapshot** we can download a .pcap file. Seems like every time we visit this page, a new file gets created. There is an IDOR vulnerability through which we can visit the page http://10.10.10.245/data/0 which was already there. If we download it and look through it, we can find captured packets from a FTP login process. We can see credentials in clear-text: 

![](/images/writeup_screens/htb_cap_1.png)

With these credentials we can access the FTP server: 

![](/images/writeup_screens/htb_cap_2.png)

We can `ls` the files and find a `user.txt`. Download the file with `get user.txt` and obtain the user flag. 

## Root Flag

* I tried if the FTP credentials may be reused for SSH and it worked.
* In the home directory we have a linpeas.sh which a can also run
* Linpeas found a binary with special capabilities:

![](../../static/images/writeup_screens/test_post.md.png)