+++
title = 'HackTheBox - Alert box write-up'
date = 2024-11-28T18:09:01+01:00
slug = "hackthebox-alert-writeup"
+++

![Alert Machine Image](/alert.png)

This is another Hack the Box machine called Alert. While gaining an initial foothold may be challenging for some (it certainly was for me), it is a super-fun machine to break into. So, here we go.

## Scanning for open ports
---

Okay, first we're going to start with some basic enumeration—we'll scan for open ports on the machine:

```
┌──(ognard㉿ognard)-[~]
└─$ nmap -sC -sV alert.htb
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-27 20:37 CET
Nmap scan report for alert.htb (10.10.11.44)
Host is up (0.046s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT  STATE SERVICE VERSION
22/tcp open ssh   OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|  3072 7e:46:2c:46:6e:e6:d1:eb:2d:9d:34:25:e6:36:14:a7 (RSA)
|  256 45:7b:20:95:ec:17:c5:b4:d8:86:50:81:e0:8c:e8:b8 (ECDSA)
|_ 256 cb:92:ad:6b:fc:c8:8e:5e:9f:8c:a2:69:1b:6d:d0:f7 (ED25519)
80/tcp open http  Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-title: Alert - Markdown Viewer
|_Requested resource was index.php?page=alert
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.14 seconds

```

We can see that only two ports are open on the machine: port **22** and port **80**.

## The Website
---

If we access http://alert.htb, we can see that the website has a couple of pages:

- The landing page has a form where we can upload and preview a Markdown file.
- The Contact page has a form where you can send a message to the owner.
- The About Us page contains some (important) information about the website.
- The Donate page is where visitors can supposedly make a donation.

Before exploring more around the website, we'll continue with some additional enumeration steps...


> ⛔
> 
> Since the Alert machine **is still active** on HackTheBox, the remainder of the write-up **will be available once the machine is retired**.
> In the meantime, for any hints or assistance, feel free to DM me on the [HackTheBox Discord Server](https://discord.gg/hackthebox?ref=ognard.com)
> or **connect with me** on [LinkedIn](https://linkedin.com/in/drangovski) and reach out to me there.




