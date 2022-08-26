## Swagshop


| Name   |      Description |  
|----------|:------------- |
| Machine IP |  10.10.10.140 |
|Difficulty |    Easy   |


## Scanning

```bash
$ nmap -sC -sV   -oA nmap/tabby  10.10.10.194
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-20 04:39 EDT
Nmap scan report for 10.10.10.194
Host is up (0.096s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 45:3c:34:14:35:56:23:95:d6:83:4e:26:de:c6:5b:d9 (RSA)
|   256 89:79:3a:9c:88:b0:5c:ce:4b:79:b1:02:23:4b:44:a6 (ECDSA)
|_  256 1e:e7:b9:55:dd:25:8f:72:56:e8:8e:65:d5:19:b0:8d (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Mega Hosting
8080/tcp open  http    Apache Tomcat
|_http-title: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.61 seconds
```

## Enumeration

- Homepage - port 80

![homepage](images/homepage.PNG)

- Homepage - port 8080

![hompage_8080](images/hompage_8080.PNG)

__Enumerating port 80__

- No link seem to work, but `news` redirected to a new page `http://megahosting.htb/news.php?file=statement`

- This seem to be vulnerable to `directory traversal`

![directory_traversal](images/directory_traversal.PNG)

__Enumerating port 8080__

- gobuster did not result in any hidden directories or files
- We are able to access `manager` login page

![manager](images/manager.PNG)

- Tried with all default `username:password`, but unable to login.
- But we can locate `tomcat-users.xml` file where `credentials` can be found.

- After enumerating, the file seem to be found in location `usr/share/tomcat9/etc/tomcat-users.xml`

![creds](images/creds.PNG)

- Now log in with found `tomcat:$3cureP4s5w0rd123!`, to upload a shell on the server