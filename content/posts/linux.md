+++
title = 'Exploring Linux Basics'
date = 2024-10-31T10:09:40+01:00
slug = "exploring-linux-basics"
+++

In this post, we’ll explore some essential Linux commands that are vital for effective management, monitoring, and configuration of files and system settings. These commands will help us to navigate and control the Linux environment with ease. 

## User Management
---

The objectives for this task were to create users and groups, set up user directories, manage permissions, and configure user-specific settings. We're going to start with creating a specific user's group called `advanced_users` with **GID** set to 1337.

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo groupadd -g 1337 advanced_students
```

In the next step, we're going to add a new user `student2`. Upon creation, we're going to set **UID** to 1001 and assign the 1337 ad the **GID** from the group we've previously created (the user will be added to `advanced_students` group based on this GID), as well as we are going to set a custom home directory for the user:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo useradd -u 1001 -g 1337 -d /home/advanced_users/student2 -m student2
```

> `-m` flag makes sure that the specified directory will be created in case it doesn't already exist.

Next, we'll apply a password expiration policy for the newly added user, so that the password will need to be changed every 30 days:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo chage -M 30 student2  
```

To check if the policy was applied, we use `sudo change -l student2` like this:
```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo chage -l student2   
Last password change                                    : Oct 30, 2024
Password expires                                        : Nov 29, 2024
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 30
Number of days of warning before password expires       : 7
```

Next, we'll set the default shell for the user to `zsh`:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo chsh -s /bin/zsh student2
```

To check if the action was successful, we can do the following:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ grep "^student2:" /etc/passwd 
student2:x:1001:1337::/home/advanced_users/student2:/bin/zsh
```

Finally, we will demonstrate the ability to lock and unlock a particular user:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo usermod -L student2  
```

To check if the user was successfully locked:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo passwd -S student2     
student2 L 2024-10-30 0 30 7 -1
```

Having `L` after the username, signifies that the user was locked. To unlock use:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo usermod -U student2             
usermod: unlocking the user's password would result in a passwordless account.
You should set a password with usermod -p to unlock this user's password.
```

> If there isn't password for the user, then it has to be set before unlocking it. Suggestion points that you can use `sudo usermod <user> -p <password>` to set the password, however you should be aware that this action may reveal the newly set password through terminal's history.



## Process Management
---

The objectives for this task are to start, monitor, and manage processes using advanced tools and techniques.

For this one, we are going to start with long running process; we will use `ping` command set to run in the background and keep look of its action:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ ping ognard.com > ping_log.txt 2>&1 &
[1] 23604
```

> The log will be stored as `ping_log.txt` and we'll catch any outputs in Standard Output (stdout) and Standard Error (stderr) with `2>&1`.

Next we will find the ongoing process by using `ps` like this:
```
┌──(ognard㉿ognard)-[~/Practice]
└─$ ps aux | grep ping                   
ognard     26267  0.0  0.0   9512  3200 pts/0    SN   20:22   0:00 ping ognard.com
```

Additionally, we can use `htop` to find the running process. Once `htop` is opened, you can hit `F3` to search for `ping` and it will show up the process information.

![htop Preview](/htop-preview.png)

Once we have found the `PID` for the running process, we can also use `renice` to change the priority of the process. For example:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo renice +10 -p 34259             
34259 (process ID) old priority 5, new priority 10
```

> For example, changing priority of the process is useful for adjustment of the resources consumption (CPU) in relation to other running processes.

Finally, to trace system calls that were made by the `ping` process we can use `strace`:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo strace -p 35193   
strace: Process 35193 attached
restart_syscall(<... resuming interrupted read ...>) = 0
sendto(3, "\10\0\211\374\211y\0\36\207\213\"g\0\0\0\0u\246\6\0\0\0\0\0\20\21\22\23\24\25\26\27"..., 64, 0, {sa_family=AF_INET, sin_port=htons(0), sin_addr=inet_addr("104.21.84.127")}, 16) = 64
recvmsg(3, {msg_name={sa_family=AF_INET, sin_port=htons(0), sin_addr=inet_addr("104.21.84.127")}, msg_namelen=128 => 16, msg_iov=[{iov_base="E\0\0TS\236\0\0:\1o\303h\25T\177\300\250@\v\0\0\221\374\211y\0\36\207\213\"g"..., iov_len=192}], msg_iovlen=1, msg_control=[{cmsg_len=32, cmsg_level=SOL_SOCKET, cmsg_type=SO_TIMESTAMP_OLD, cmsg_data={tv_sec=1730317191, tv_usec=449569}}], msg_controllen=32, msg_flags=0}, 0) = 84
...
```



## Practical Usage of Commands
---

The objectives for this task is to use a variety of system commands to perform complex tasks. We'll start with creating a directory structure and setting appropriate permissions. First, to create directories:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ mkdir dir_parent            
                           
┌──(ognard㉿ognard)-[~/Practice]
└─$ mkdir dir_parent/dir_child_1
                      
┌──(ognard㉿ognard)-[~/Practice]
└─$ mkdir dir_parent/dir_child_2
                  
┌──(ognard㉿ognard)-[~/Practice]
└─$ mkdir dir_parent/dir_child_3
```

> The shorthand way to do this is `mkdir -p parent_dir/{child_1,child_2,child_3}`

To preview the structure, we can use `tree` command:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ tree dir_parent         
dir_parent
├── dir_child_1
├── dir_child_2
└── dir_child_3
```

To change permissions for `dir_parent` directory, we can use:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo chmod 644 dir_parent  
```

> Cheatsheet: **Read** is **4**, **Write** is **2** and **Execute** is **1**.

This will set the permissions to `read and write` for the **owner** and `read` for the **group** and **global** . To verify the permissions, after `ls -al` we can see:

```
drw-r--r--  5 ognard ognard 4096 Oct 30 20:55 dir_parent
```

- **rw-** for the owner
- **r--** for the group
- **r--** for global

Next coming up is copying files from one directory to another. First, for the purpose of this task, we will create an example file in `dir_parent/dir_child_1`:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ echo "Some text for the example file 1" > dir_parent/dir_child_1/example_file
```

After that, we will proceed with copying the file to another directory:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ cp dir_parent/dir_child_1/example_file dir_parent/dir_child_2
```

To verify the copying process, we can use `tree`:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ tree dir_parent       
dir_parent
├── dir_child_1
│   └── example_file
├── dir_child_2
│   └── example_file
└── dir_child_3
```

And we can see that the same file is in both folders - `dir_child_1` and `dir_child_2`. Alternatively, we can navigate to the destination directory and check if the file is copied there as well.

Next is using `rsync` to synchronize directories and preserve permissions:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ rsync -av --progress ./src_dir ./dst_dir 
sending incremental file list
src_dir/

sent 75 bytes  received 20 bytes  190.00 bytes/sec
total size is 0  speedup is 0.00

```

- **-av** will set `Archive Mode` and `Verbose Output`. Archive Mode will preserve permissions.
- **--progress** will display progress during the transfer.

Another objective is to securely transfer files to a remote server using `scp` :

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ scp ./example_file ognard@[redacted]:~/
ognard@[redacted]'s password: 
example_file                              100%   43     1.2KB/s   00:00 
```

To verify the transfer, we can connect through ssh on the remote server and check the destination directory:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ ssh ognard@[redacted]
ognard@[redacted]'s password: 
ognard@ubuntu-s-1vcpu-512mb-10gb-fra1-01:~$ ls -al
total 32
drwxr-x--- 3 ognard ognard 4096 Oct 30 21:22 .
drwxr-xr-x 3 root   root   4096 Oct 30 21:19 ..
-rw------- 1 ognard ognard   18 Oct 30 21:22 .bash_history
-rw-r--r-- 1 ognard ognard  220 Oct 30 21:19 .bash_logout
-rw-r--r-- 1 ognard ognard 3771 Oct 30 21:19 .bashrc
drwx------ 2 ognard ognard 4096 Oct 30 21:20 .cache
-rw-r--r-- 1 ognard ognard    0 Oct 30 21:19 .cloud-locale-test.skip
-rw-r--r-- 1 ognard ognard  807 Oct 30 21:19 .profile
-rw-rw-r-- 1 ognard ognard   43 Oct 30 21:22 example_file

```

Finally, the last bit of the task is to check for open ports and active connections with `netstat`:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ netstat -tuln          
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:5355            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN     
tcp6       0      0 :::5355                 :::*                    LISTEN     
udp        0      0 0.0.0.0:5353            0.0.0.0:*                          
udp        0      0 0.0.0.0:5355            0.0.0.0:*                          
udp        0      0 127.0.0.54:53           0.0.0.0:*                          
udp        0      0 127.0.0.53:53           0.0.0.0:*                          
udp6       0      0 :::5353                 :::*                               
udp6       0      0 :::5355                 :::*                               
```



## Package Management
---

Next task objectives are to install, remove, and manage packages using dpkg and apt, and handle package dependencies.

We will start with updating the package lists and upgrading all installed packages:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo apt update && upgrade
Hit:1 http://http.kali.org/kali kali-rolling InRelease
1645 packages can be upgraded. Run 'apt list --upgradable' to see them.
...
```

Next, as an example for using `dpkg` to install a package we can do:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo dpkg -i cowsay-off_3.03+dfsg1-15_all.deb 
Selecting previously unselected package cowsay-off.
(Reading database ... 396660 files and directories currently installed.)
Preparing to unpack cowsay-off_3.03+dfsg1-15_all.deb ...
Unpacking cowsay-off (3.03+dfsg1-15) ...
dpkg: dependency problems prevent configuration of cowsay-off:
 cowsay-off depends on cowsay (>= 3.03+dfsg1-13); however:
  Package cowsay is not installed.

dpkg: error processing package cowsay-off (--install):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 cowsay-off
                                                                                                                                                            
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo apt install -f                          
Correcting dependencies... Done 
The following packages were automatically installed and are no longer required:
  ibverbs-providers         libcephfs2  libgfxdr0      libpython3.11-dev  python3-lib2to3  python3.11-minimal
  libboost-iostreams1.83.0  libgfapi0   libglusterfs0  librados2          python3.11       samba-vfs-modules
  libboost-thread1.83.0     libgfrpc0   libibverbs1    librdmacm1t64      python3.11-dev
Use 'sudo apt autoremove' to remove them.

Installing dependencies:
  cowsay

Suggested packages:
  filters

Summary:
  Upgrading: 0, Installing: 1, Removing: 0, Not Upgrading: 1646
  1 not fully installed or removed.
  Download size: 21.4 kB
  Space needed: 94.2 kB / 44.9 GB available

Continue? [Y/n] y
Get:1 http://http.kali.org/kali kali-rolling/main amd64 cowsay all 3.03+dfsg2-8 [21.4 kB]
Fetched 21.4 kB in 0s (69.0 kB/s)  
Selecting previously unselected package cowsay.
(Reading database ... 396668 files and directories currently installed.)
Preparing to unpack .../cowsay_3.03+dfsg2-8_all.deb ...
Unpacking cowsay (3.03+dfsg2-8) ...
Setting up cowsay (3.03+dfsg2-8) ...
Setting up cowsay-off (3.03+dfsg1-15) ...
Processing triggers for man-db (2.12.1-2) ...

```

> Since there were some dependency issues, we used `sudo apt install -f` to satisfy the needs.

To hold a package, we can do:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ echo "cowsay hold" | sudo dpkg --set-selections
```

To check which packages are held:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ dpkg --get-selections | grep hold
cowsay                                          hold
```

To search and show details for a package:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ apt search cowsay       
cowsay/kali-rolling,now 3.03+dfsg2-8 all [installed,automatic]
  configurable talking cow

cowsay-off/kali-rolling 3.03+dfsg2-8 all [upgradable from: 3.03+dfsg1-15]
  configurable talking cow (offensive cows)

presentty/kali-rolling 0.2.1-1.1 all
  Console-based presentation software

xcowsay/kali-rolling 1.6-1+b1 amd64
  Graphical configurable talking cow

...

┌──(ognard㉿ognard)-[~/Practice]
└─$ apt search cowsay       
cowsay/kali-rolling,now 3.03+dfsg2-8 all [installed,automatic]
  configurable talking cow

cowsay-off/kali-rolling 3.03+dfsg2-8 all [upgradable from: 3.03+dfsg1-15]
  configurable talking cow (offensive cows)

presentty/kali-rolling 0.2.1-1.1 all
  Console-based presentation software

xcowsay/kali-rolling 1.6-1+b1 amd64
  Graphical configurable talking cow

```

Finally to remove a package, we can do the following:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo apt remove cowsay         
The following packages were automatically installed and are no longer required:
  ibverbs-providers         libcephfs2  libgfxdr0      libpython3.11-dev  python3-lib2to3  python3.11-minimal
  libboost-iostreams1.83.0  libgfapi0   libglusterfs0  librados2          python3.11       samba-vfs-modules
  libboost-thread1.83.0     libgfrpc0   libibverbs1    librdmacm1t64      python3.11-dev
Use 'sudo apt autoremove' to remove them.

Changing held packages:
  cowsay

REMOVING:
  cowsay  cowsay-off

Summary:
  Upgrading: 0, Installing: 0, Removing: 2, Not Upgrading: 1645
  Freed space: 118 kB

Continue? [Y/n] y
(Reading database ... 396729 files and directories currently installed.)
Removing cowsay-off (3.03+dfsg2-8) ...
Removing cowsay (3.03+dfsg2-8) ...
Processing triggers for man-db (2.12.1-2) ...
                                                                                                                                                            
┌──(ognard㉿ognard)-[~/Practice]
└─$ sudo apt purge cowsay 
Package 'cowsay' is not installed, so not removed
The following packages were automatically installed and are no longer required:
  ibverbs-providers         libcephfs2  libgfxdr0      libpython3.11-dev  python3-lib2to3  python3.11-minimal
  libboost-iostreams1.83.0  libgfapi0   libglusterfs0  librados2          python3.11       samba-vfs-modules
  libboost-thread1.83.0     libgfrpc0   libibverbs1    librdmacm1t64      python3.11-dev
Use 'sudo apt autoremove' to remove them.

Summary:
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 1645

```



## Network Troubleshooting
---

The objectives of this task are to diagnose and fix network issues using various tools. First we'll start with using `ping` to check connectivity with sending only **4** packets by using `-c` flag:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ ping -c 4 ognard.com
PING ognard.com (172.67.193.6) 56(84) bytes of data.
64 bytes from 172.67.193.6: icmp_seq=1 ttl=50 time=46.9 ms
64 bytes from 172.67.193.6: icmp_seq=2 ttl=50 time=46.9 ms
64 bytes from 172.67.193.6: icmp_seq=3 ttl=50 time=47.2 ms
64 bytes from 172.67.193.6: icmp_seq=4 ttl=50 time=46.5 ms

--- ognard.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3010ms
rtt min/avg/max/mdev = 46.471/46.846/47.167/0.248 ms

```

To trace the route to the remote server we can use `traceroute`. For example:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ traceroute ognard.com
traceroute to ognard.com (104.21.84.127), 30 hops max, 60 byte packets
 1  _gateway (192.168.64.1)  1.930 ms  1.629 ms  0.945 ms
 2  192.168.1.1 (192.168.1.1)  2.604 ms  2.558 ms  2.511 ms
 3  62.162.200.200 (62.162.200.200)  7.537 ms  7.340 ms  7.190 ms
 4  62.162.201.1 (62.162.201.1)  7.140 ms  7.093 ms  7.045 ms
 5  * * 95.158.152.41 (95.158.152.41)  7.575 ms
 6  * * *
 7  185.1.40.27 (185.1.40.27)  20.070 ms  10.072 ms  10.004 ms
 8  104.21.84.127 (104.21.84.127)  7.882 ms  9.722 ms  9.625 ms
```

> To identify bottlenecks, we can look for significant increases in the response time for any of the hops, which can suggest that there is a network congestion. Also, as we may see from the example above, if some of the hops show `***`  that suggests that there may be some issues at that hop, such as packet drops or filtering.

Next objective is to check the configuration of network interfaces. We can do that with `ip a` or `ifconfig`:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ ip a         
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 4e:81:77:5a:e0:ef brd ff:ff:ff:ff:ff:ff
    inet 192.168.64.11/24 brd 192.168.64.255 scope global dynamic noprefixroute eth0
       valid_lft 3005sec preferred_lft 3005sec
    inet6 fdfc:e305:f06b:6649:f343:140a:983a:5af3/64 scope global temporary dynamic 
       valid_lft 604208sec preferred_lft 85458sec
    inet6 fdfc:e305:f06b:6649:4c81:77ff:fe5a:e0ef/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 2591964sec preferred_lft 604764sec
    inet6 fe80::4c81:77ff:fe5a:e0ef/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

Finally, we can use `netstat` to display network connections, routing tables and interface statistics. For example:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ netstat -tuln          
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:5355            0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN     
tcp6       0      0 :::5355                 :::*                    LISTEN     
udp        0      0 0.0.0.0:5353            0.0.0.0:*                          
udp        0      0 0.0.0.0:5355            0.0.0.0:*                          
udp        0      0 127.0.0.54:53           0.0.0.0:*                          
udp        0      0 127.0.0.53:53           0.0.0.0:*                          
udp6       0      0 :::5353                 :::*                               
udp6       0      0 :::5355                 :::*                               
```

> Flags used are `-t` to display TCP, `-u` to display UDP, `-l` to display listening and `-n` to display numerical connections. These flags are helpful for seeing open ports, active connections and statistics.

To see the routing tables, we can use `-r` flag:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         _gateway        0.0.0.0         UG        0 0          0 eth0
192.168.64.0    0.0.0.0         255.255.255.0   U         0 0          0 eth0
```

And to display interface statistics, we can use `-I` flag:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ netstat -i
Kernel Interface table
Iface             MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
eth0             1500       95      0      0 0           139      0      0      0 BMRU
lo              65536       88      0      0 0            88      0      0      0 LRU
```



## Files Manipulation
---

The objectives for the final task are to perform complex file manipulation tasks, including searching, sorting and editing files. We'll start with generating a file that will be used for this purpose:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ seq 1 1000 | shuf > 1000_random_numbers.txt
```

> `seq` will generate numbers from 1 to 1000, one number per line and `shuf` will shuffle, or randomize the order of the lines. After that we will store the output to a file.

We will modify the last lines of the file, just for the purpose of the upcoming objectives and making the process more clear. To do that we can use `nano` and just add the last three (620) lines like this:

```
...
70
300
18
620
620
620
620
This line will serve for grep objective.
```

Now, to search for specific patters in this file, we can use `grep`:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ grep "objective" 1000_random_numbers.txt
This line will serve for grep objective.
```

> This will show very line where the pattern (i.e. "objective") exists.

Next we will use `awk` for processing and formatting the file. For example, we can set a prefix for each line so that it will mark the line number and store it to a new file:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ awk '{print "Line " NR ": " $0}' 1000_random_numbers.txt > prefixed_1000_random_numbers.txt
```

Sample of the newly generated file:

```
Line 998: 300
Line 999: 18
Line 1000: 620
Line 1001: 620
Line 1002: 620
Line 1003: 620
Line 1004: This line will serve for grep objective.
```

For the next objective, we will sort the contents and remove the duplicates in the original file:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sort -n 1000_random_numbers.txt| uniq > sorted_1000_random_numbers.txt
```

`sort` will organize the lines in ascending order by default and `uniq` will remove duplicate lines (this only works if the file is already sorted), and finally it will store the output to a new file.

> `-n` flag for `sort` will set the sorting in numeric order instead of alphabetical.

Example output:

```
989
990
991
992
993
994
995
996
997
998
999
1000
```

And if we use `grep` again to search for the duplicate **620**, we can see that there are no more duplicates of it:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ grep "620" sorted_1000_random_numbers.txt 
620
```

Next objective is to perform substitution in the file. For this purpose we will use the text line we've added earlier in the file:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ sed 's/grep/substitution/g' 1000_random_numbers.txt
...
620
620
This line will serve for substitution objective.
```

> Note that the word "grep" was changed to "substitution". Of course, we could have stored this output to a new file as well.

And finally, the last objective is to compress the text file using `gzip`:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ gzip 1000_random_numbers.txt 
```

And then we can decompress the file:

```
┌──(ognard㉿ognard)-[~/Practice]
└─$ gunzip 1000_random_numbers.txt.gz  
```
