---
title: overthewire&#58; natas 0 - 5
tags: web writeup
---

## Natas 0

Username: `natas0`

Password: `natas0`

Url: `http://natas0.natas.labs.overthewire.org/`

We can View Source and find the next password as a html comment

![](images/Pasted image 20210312152338.png)

___

## Natas 1
Username: `natas1`

Password: `gtVrDuiDfck831PqWsLEZy5gyDz1clto`

Url: `http://natas1.natas.labs.overthewire.org/`

The same as before the password is html source but we can not right click on the web page, we can use `CTRL+U` or `CTRL+SHIFT+I` to bypass

![](images/Pasted image 20210312152438.png)

___

## Natas 2

Username: `natas2`

Password: `ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi`

Url: `http://natas2.natas.labs.overthewire.org/`

By viewing the source we can find an image

![](images/Pasted image 20210312152516.png)

which is in `/files` 

There is a users.txt file here

![](images/Pasted image 20210312152559.png)

Which contains the password for natas3

___ 

##  Natas 3

Username: `natas3`

Password: `sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14`

Url: `http://natas3.natas.labs.overthewire.org/`

After looking at the source code we get a hint

![](images/Pasted image 20210312152715.png)

This hints we can check for a `robots.txt`
In robots.txt we find a directory: `/s3cr3t` 
this directory has a users.txt file where we can find the password for natas4 

![](images/Pasted image 20210312152836.png)

___

## Natas 4

Username: `natas4`

Password: `Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ` 

Url: `http://natas4.natas.labs.overthewire.org/`

By going to natas4 we see:

![](images/Pasted image 20210312153242.png)

By analying out request with burp

![](images/Pasted image 20210312160113.png)

Here we understand that we need to change our `Referer:` header to match as if we were coming from `http://natas5.natas.labs.overthewire.org/`. Changing our referer value to the above url gets us the next password.

![](images/Pasted image 20210312160129.png)

___

## Natas 5

Username: `natas5`

Password: `iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq`

Url: `http://natas5.natas.labs.overthewire.org/`

By visting the site we get an `Access disallowed. You are not logged in `

We can check the cookies to see how the web page handles our session

We notice there is a cookie `loggedin-0` lets change that to a 1. Now we can see the password for natas6.

