# DX1: Liberty Island
--------------------------------------------------------------------

## GIVEN INFO


**IP Address**: 10.10.190.59

--------------------------------------------------------------------

## PROCEDURE

```
nmap -sV -oN nmap.txt 10.10.190.59
```

**EXPOSED PORT (SERVICE)**:<br>
    80 (http Apache httpd 2.4.41),<br>
    5901 (VNC protocol 3.8)

```

Found potential username on website:<br>
AJacobson//UNATCO.00013.76490

dirb http://10.10.190.59 /usr/share/wordlists/dirb/common.txt
```

Found hidden directory (/datacubes) in /robots.txt and name "alex"

```
dirb http://10.10.190.59/datacubes /usr/share/wordlists/dirb/common.txt
```

http://10.10.190.59/datacubes goes to http://10.10.190.59/datacubes/0000 so we can try fuzzing the last value<br>
-w will put leading zeroes<br>
```
seq -w 0000 9999 > nums.txt
wfuzz -c -z file,nums.txt --hc 404 http://10.10.190.59/datacubes/FUZZ 
```

Valid Values: 0000, 0011, 0068, 0103, 0233, 0451

Found name "Jonathan" in 0068<br>
Found username "ghermann" in 0103<br>
From 0451, vnc setup under "jacobson" account<br>

VNC login: 'smashthestate', hmac'ed with my username from the 'bad actors' list
Use md5 for the hmac hashing algo. The first 8 characters of the final hash is the VNC password. - JL


Bad Actors List:<br>
apriest<br>
aquinas_nz<br>
cookiecat<br>
craks<br>
curley<br>
darkmattermatt<br>
etodd<br>
gfoyle<br>
grank<br>
gsyme<br>
haz<br>
hgrimaldi<br>
hhall<br>
hquinnzell<br>
infosneknz<br>
jallred<br>
jhearst<br>
jlebedev<br>
jooleeah<br>
juannsf<br>
killer_andrew<br>
lachland<br>
leesh<br>
levelbeam<br>
mattypattatty<br>
memn0ps<br>
nhas<br>
notsus<br>
oenzian<br>
roseycross<br>
sjasperson<br>
sweetcharity<br>
tfrase<br>
thom_seven<br>
ttong<br>search 

jlebedev

baf85b09178098a8e19921accd98d19d

baf85b09

```
msfconsole
search vnc
use auxillary/scanner/vnc_login

```
