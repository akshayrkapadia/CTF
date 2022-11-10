10.10.236.122

```
nmap -sV -p- 10.10.236.122
```

![1.1](./imgs/1.1.png)

Webserver on port 80 is default apache page

ftp has anonymous login enabled

```
ftp anonymous@10.10.236.12
ls -la
```

![1.2](./imgs/1.2.png)

```
less .info.txt
```

![1.3](./imgs/1.3.png)

Possible substituion cipher<br>
**TEXT**: Whfg jnagrq gb frr vs lbh svaq vg. Yby. Erzrzore: Rahzrengvba vf gur xrl!

Try accessing http server at http://10.10.236.122:10000

![1.4](./imgs/1.4.png)

Need to add new host to /etc/hosts
```
echo "10.10.236.122 ip-10-10-236-122.eu-west-1.compute.internal" >> /etc/hosts
```

![1.5](./imgs/1.5.png)

webmin login page found

![1.6](./imgs/1.6.png)

Enumerate hidden webdirectories on port 80
```
feroxbuster -u http://10.10.236.122 -w /usr/share/wordlists/dirb/common.txt 
```

Found robots.txt

![1.7](./imgs/1.7.png)




### CRACK CIPHER

