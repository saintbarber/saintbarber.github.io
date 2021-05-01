---
title: CCSC 2021&#58; Cate Bianca
tags: ccsc web writeup
---

## Solution

This challenge was a simple web application with a login and registeration system. We could use SQL Truncation attack to overwrite the admin users password and login as admin to get to the flag

___

## Description

This is a website dedicated to an exceptional player who originates from a small village in Switzerland called Trun. Cate was a sensational player whose career was shortened by the weaknesses of her vendors and ultimately led to her defeat in the 2018 U.S. Chess Championship semifinals.

___

Well from the challenge description i noticed the wordplay (Trun. Cate -> Truncate) which made me believed it was an sql truncation attack from the beginning, but i still did some standard tests to the web application beforehand.

Once we open the web application i was greeted with a simple home page and two links, a Login and Register link.

![](images/Pasted%20image%2020210501184809.png)

The register page only requiered a username and password, i quickly created one with credentials `saint:saint` and logged in.

![](images/Pasted%20image%2020210501184932.png)

Once i logged in i was redirected to an `account.php` page

![](images/Pasted%20image%2020210501185011.png)

Here i though about SSTI and XSS since it renders my username, i also tested the login and register page for sql injections but nothing came up. Next i tried to register a user with a the name `admin` and got a `Username is taken` error.

![](images/Pasted%20image%2020210501185129.png)

Before the CTF started all competitors knew that each challenge were seperate to each player, meaning that i know the admin user account was not taken by another competitor and it was part of the challenge.

Now i know i need to somehow login to the admin user, but i do not know the password and there is no sql injection on either page.

Another thing i noticed is at the register page the username and password fields only accepts up to 20 characters

![](images/Pasted%20image%2020210501185457.png)

We can see this by viewing the source code of the register page. `maxlength="20"`

![](images/Pasted%20image%2020210501185708.png)

Even if someone did not notice the wordplay in the description after all this information on the web app the attack becomes quite obvious.

___

## SQL Truncation Attack

A SQL Truncation Attack is when the database expects a certain number of characters for an individual field for example if we create a database with the folowing script:

```sql
create table users(
   id INT NOT NULL AUTO_INCREMENT,
   username VARCHAR(20) NOT NULL,
   password VARCHAR(20) NOT NULL,
   PRIMARY KEY ( user_id )
);
```

 Meaning if i provide a `username` larger than 20 characters the rest will be truncated, for example the username `reallylongusername1234saintbarber` will become `reallylongusername12`
 
 Also trailing spaces are considered equal in mysql, for example the username `"admin"` is equal to `"admin "`, but we could not just register `"admin "` (with a space) in the web app as it is equal to `"admin"`  which is already taken. 

Combining these two identities we get the SQL Truncation Attack, one can register a username that is larger that 20 character with trailing spaces and some random characters at the end, the characters at the end will be truncated and the trailing spaces will remain, since there are random chars at the end the database will interpret the username as a seperate entry, ultimatly overwriting the admin password since it is considered the same.

For example if i register a username equal to `adminxxxxxxxxxxxxxxxrandomchar` (the `x`'s are spaces), `randomchar` will be truncated leaving us with `adminxxxxxxxxxxxxxxx` which is equal to `admin`. Because we have the random characters at the end it will be considered as a sperate entry as because  `adminxxxxxxxxxxxxxxxrandomchar` is not equal to `admin`.

Meaning we can change the password and login to the admin account.

The developer of the website used the `maxsize` attribute to protect the site from this attack on the client side, but did not consider performing the same check server side, meaning i could just intercept the request with a program like burp suite and provide a username with a larger length than 20 characters

![](images/Pasted%20image%2020210501214610.png)

Next i logged in with `admin:password` and got the flag


![](images/Pasted%20image%2020210501214655.png)

Flag: `CCSC{7runc473_y0ur_w4y_70_6r4ndm4573r}`
 