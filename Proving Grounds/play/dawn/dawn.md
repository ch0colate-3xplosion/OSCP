### Dawn

Machine difficulty: Easy
IP: 192.168.84.11

## Scanning

### Nmap Scanning

```bash
$ nmap -sC -sV -oA nmap/initial 192.168.84.11                             1 ⚙
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-15 13:12 EDT
Nmap scan report for 192.168.84.11
Host is up (0.10s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Site doesn't have a title (text/html).
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL 5.5.5-10.3.15-MariaDB-1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.15-MariaDB-1
|   Thread ID: 14
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, Speaks41ProtocolNew, DontAllowDatabaseTableColumn, SupportsTransactions, IgnoreSigpipes, Speaks41ProtocolOld, InteractiveClient, ConnectWithDatabase, FoundRows, LongColumnFlag, ODBCClient, SupportsLoadDataLocal, IgnoreSpaceBeforeParenthesis, SupportsCompression, SupportsMultipleResults, SupportsMultipleStatments, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: U0/=#8.~QZO]d_0oaxX.
|_  Auth Plugin Name: mysql_native_password
Service Info: Host: DAWN

Host script results:
|_clock-skew: mean: 1h20m00s, deviation: 2h18m34s, median: 0s
|_nbstat: NetBIOS name: DAWN, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: dawn
|   NetBIOS computer name: DAWN\x00
|   Domain name: dawn
|   FQDN: dawn.dawn
|_  System time: 2021-08-15T13:13:16-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-08-15T17:13:16
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.00 seconds
```
