## Postman


| Name   |      Description |  
|----------|:------------- |
| Machine IP |  10.10.10.160 |
|Difficulty |    Easy   |

## Scanning

### Nmap scanning

```bash
```


## Enumeration

__Website__

- The site is empty and nothing interesting in it.

![homepage](images/homepage.PNG)

### Gobuster

```bash
$ gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt dir -u  http://10.10.10.160 -o gobuster/gobuster.out  
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.160
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/09/17 02:57:11 Starting gobuster
===============================================================
/images (Status: 301)
/upload (Status: 301)
/css (Status: 301)
/js (Status: 301)
/fonts (Status: 301)
Progress: 16477 / 220561 (7.47%)^C
[!] Keyboard interrupt detected, terminating.
===============================================================
2021/09/17 03:00:07 Finished
```


- Nothiong interesting in `gobuster` out.

__Webadmin - TCP 10000__

There is a webadmin instance on port 10000, but returns error when visited.

![webadmin](images/webadmin.PNG)

Found a domain name `Postman`. and this to `etc/hosts` file.

![postman_login](images/postman_login.PNG)

__Exploring Redis - TCP 6379__

Interacting `redis` with `nc`

```bash
$ nc 10.10.10.160 6379                      
keys *
*1
$7
ssh_key
get ssh_key
$568


ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC26uS8H6Ilce/SVT4BO3mYFwmlYWrLrPczV4LiMZdx8YSxJ1ugwJIVXdQIXZxdk63SPbHVrcJX5HnE7jW3qCOp6HPSvaKUGXwjyxoAf9A+FJ52JMId7UbJztTM3z/6QBpYT0yRVXgcbX/GcA3VnwA9tUbMaJ8TfWu5/xA5CCnoMv1CRXYjGo0u0QWMwoPfDXi3p6LIfCYYAuVWE14CvfY910gzZBWoH5tvKV6ceWwWF2swhjlb6toBv+z1iugOCrUcVZTP2+8iYh37l5IL9pk3N1Aqws9rppkucSzxVeKE+jeBlhptbwCOwEaen2YIy6cOt8Y1vWvUD7sOv9SULQelkRxYwLIEgwlrmJXzmhDJ5++EJZ0Yn0Loo2ISFcHbjrjCODzgjOv6X6p8lewN2NjQNrefAcehSOf1YXcA79t05HFiSiWNOPs78+K96ovi67DCeEXuizlTFAr6iG2L0iMr3yMugd8aGNDiOAEN/yqAQ0pPfRNaaZFUL3NyfEzt78U= root@kali

```

- Found an `ssh` key, but thats not intented way.

- Injecting `webshell` using `redis`

