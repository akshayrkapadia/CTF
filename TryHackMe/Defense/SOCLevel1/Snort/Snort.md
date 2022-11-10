# SNORT

Snort is an NIDS/NIPS<br>
Its config files are in 
```
/etc/snort
```

## VALIDATE CONFIG

-V: version
-q: quiet mode<br>
-T: self test<br>
-c CONFIG_FILE<br>
```
sudo snort -T -c /etc/snort/snort.conf -q
```

## SNIFFER MODE

Similar to tcpdump

-v: verbose<br>
-i INTERFACE
```
sudo -v -i eth0
```

![1.1](./imgs/1.1)

-d: display packet data<br>
```
sudo -d -i eth0
```

![1.2](./imgs/1.2)

-e: display link-layer headers<b>
```
sudo -de -i eth0
```

![1.3](./imgs/1.3)

-X: display full packet details in HEX<br>
```
sudo -X -i eth0
```

![1.4](./imgs/1.4)


## PACKET LOGGER MODE

Log files created will be owned by root so to access the you can change ownership<br>
```
sudo chown USER LOGFILE
sudo chown USER -R LOGDIR
```

This will create a binary format log and then read it using snort<br>
-l OUTPUT_DIR (default is /var/log/snort)<br>
-r: reading option<br>
```
sudo snort -dev -l .
sudo snort -r SNORT_LOG
```

You can also filter using the -r flag<br>
-n PACKETS (number of packets to process)
```
sudo snort -r SNORT_LOG icmp -n 10
sudo snort -r SNORT_LOG "udp and port 53"
```

To view full packet details<br>
```
sudo snort -r SNORT_LOG -X
```

This will create a human readable log<br>
-K ASCII (log packets in ASCII)<br>
```
sudo snort -dev -K ACSII -l .
```

## IDS/IPS MODE

### IDS

-N: disable logging<br>
-D: background mode<br>
-A ALERT_MODE (full, fast, console, cmg, none)<br>
```
sudo snort -c /etc/snort/snort.conf -A console
```

![2.1](./imgs/2.1)

full and fast don't give console output but instead generate alert log file

```
sudo snort -c /etc/snort/snort.conf -A cmg
```

![2.2](./imgs/2.2)

### IPS

-q -Q --daq afpacket (activate IPS<br>
-i eth0:eth1 (this mode needs 2 interfaces)<br>
```
sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A console
```

## PCAP INVESTIGATION MODE

-r/--pcap-single=PCAP_FILE<br>
--pcap-list="PCAP_FILES"<br>
--pcap-show: show pcap name in console
```
sudo snort -c /etc/snort/snort.conf -q -r PCAP_FILE -A console -n 10
```

![3.1](./imgs/3.1)

## SNORT RULES

Stored in /etc/snort/rules/local.rules

![4.1](./imgs/4.1)

### Specifying IP
```
IP
IP,IP
IP/24
[IP/24, IP/24]
```

### Specifying Ports
```
PORT
:PORT (<= PORT)
PORT: (>= PORT)
MIN_PORT:MAX_PORT
PORT1, PORT2
```

### RULE OPTIONS

General Options:<br>
msg: MESSAGE;<br>
sid: RULE_IDl;<br>
referense: CVE<br>
rev: REVISION<br>

Payload Detection Rule Options:<br>
content: PAYLOAD_DATA;<br>
nocase; case insensitive<br>
fast_pattern; Select which content option to use for initial match<br>

Alert when HTTP packet has GET request
```
alert tcp any any <> any 80 (msg: "GET request found"; content: "GET"; fast_pattern; content: "www"; nocase; sid 1000001l rev: 1;) # ASCII
alert tcp any any <> any 80 (msg: "GET request found"; content: "|47 45 54|"; sid 1000001l rev: 1;) # HEX
```

Non-Payload Detection Rule Options:<br>
id: IP_ID;<br>
flags: FLAG; F-FIN, S-SYN, A-ACK, R-RST, P-PSH, U-URG<br>
dsize: PAYLOAD_SIZE; min<>max, >SIZE, \<SIZE <br>
sameip;<br>

```
alert tcp any any <> any 80 (msg: "testing"; flags: A; dsize: >100; sid 1000001l rev: 1;)
```