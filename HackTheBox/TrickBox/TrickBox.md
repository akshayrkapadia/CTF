# TRICK BOX

--------------------------------------------------------------------

**TOOLS USED**: nmap

**nmap**: network exploration tool and security / port scanner<br>

```
nmap [Scan Type...] [Options] {target specification}
```

Scan Types:<br>
-Ss [DEFAULT] TCP SYN Scan (Doesn't open full TCP connection)<br>
-St TCP Scan (Opens full TCP connection)<br>
-Su UDP Scan<br>

**gobuster**: tool to brute force URIs (dirs, files, dns subdomains)

--------------------------------------------------------------------

## GIVEN INFO

**IP Address**: 10.10.11.166

--------------------------------------------------------------------

## PROCEDURE

### 1. FIND EXPOSED PORTS

```
nmap -sC -sV -p- -oN nmap.txt 10.10.11.166
```

![nmap](./imgs/nmap.png)

**EXPOSED PORT (SERVICE)**:<br>
22 (ssh), <br>
25 (smtp), <br>
53 (domain), <br>
80 (http)
