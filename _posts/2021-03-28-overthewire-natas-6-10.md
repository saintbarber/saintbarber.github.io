---
title: overthewire&#58; natas 6 - 10
tags: web writeup
---

## Natas6
Username: `natas6`

Password: `aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1`

Url: `http://natas6.natas.labs.overthewire.org/`

By viewing the page we see a simple text field

![](images/Pasted image 20210312160424.png)

submiting anything to the field gives a `Wrong secret` message

There is a view source link as well, lets check the source code

This is the php code we get from the source code

![](images/Pasted image 20210312160543.png)

Here we see that it checks our POST parameter `secret` with the variable `$secret`, also it includes a page from `includes/secret.inc`. The `$secret` variable must be from there. We can navigate to the `includes/secret.inc` and see if we can view its source.

![](images/Pasted image 20210312160853.png)

And we get the secret: `FOEIUWGHFEEUHOFUOIU`

Inputing this secret gets us the password for natas7

___
## Natas7
Username: `natas7`

Password: `7z3hEENjQtflzgnT29q7wAvMNfZdh0i9`

Url: `http://natas7.natas.labs.overthewire.org/`

Here we have a simple page with two links. By clicking one of the links we notices it fetches the page with the GET parameter `page`. Test for LFI here by changing the parameter to `/etc/passwd`

![](images/Pasted image 20210312161034.png)

As we can see below we have Local File Inclusion. Now lets try and find a file that conatains the password to `natas8`

![](images/Pasted image 20210312161141.png)

We can note from the home page of the Natas web challenges

![](images/Pasted image 20210312162412.png)

all passwords are stored in `/etc/natas_webpass/natasX`

So we can assume that the natas7 challenge is running as natas7 and we can read natas8

url: `/index.php?page=/etc/natas_webpass/natas8`

![](images/Pasted image 20210312162536.png)

___
## Natas8
Username: `natas8`

Password: `DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe`

Url: `http://natas8.natas.labs.overthewire.org/`

We notice that this site looks alot like natas6, lets check the source code again

![](images/Pasted image 20210312162724.png)

Looks like the secret is encrypted. The web server will take our input, encrypt it with the `encodedSecret` function and then check if it is equal to the `$encodedSecret` variable. 

First is will `base64 encode` -> `Reverse string` -> `Convert to hex`

Let do the opposite to the string we already have

`Convert from hex` -> `reverse string` -> `base64 decode`

Ill be using a tool called cyber chef to do this (https://gchq.github.io/CyberChef/)

And we get our secret

![](images/Pasted image 20210312163104.png)

By submiting `oubWYf2kBq` we get the password for natas9

___
## Natas9
Username: `natas9`

Password: `W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl`

Url: `http://natas9.natas.labs.overthewire.org/`

By viewing the page we get an input box and a view source code link

![](images/Pasted image 20210312164335.png)

By submiting anything it looks like it searches for the word and print the output, lets look at the source code

![](images/Pasted image 20210312164432.png)

As we can see from the php code it takes our input and uses the `passthru` function to grep the dictionary.txt file. 

By reading online we understand that the passthru function performs OS commands, our `$key` variable is not sanitized meaning we can injection malicious commands.

`grep -i $key dictionary.txt`

We can end a command and add more commands in one line in bash with the `;`

We can also comment bash code with a `#`

if we inject a command like: `; cat /etc/passwd #` we should exeute the `cat /etc/passwd` command and view the /etc/passwd file:

![](images/Pasted image 20210312164853.png)

bingo, we have command execution, lets try and read the `natas10` password file with: `; cat /etc/natas_webpass/natas10 #`

![](images/Pasted image 20210312165048.png)

___
## Natas10
Username: `natas10`

Password: `nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu`

Url: `http://natas10.natas.labs.overthewire.org/`

We get the same challenge as before, but now it states that they filter the key we supply. Lets check the source code

![](images/Pasted image 20210312165949.png)

As we can see it checks that our input does not conatain `;`,`|` or `&`. So we can not use the same code injection bypass as we did before, as the `;` is not allowed. But the `#` is still allowed, meaning me can remove the `dictionary.txt` from the grep command as we did before and supply our own file to read from.

What if we supplied the `/etc/natas_webpass/natas11` file? since we know that grep will print the whole line if it contains as least one char that matches.

Lets try `a /etc/natas_webpass/natas11 #` -> nope

`b /etc/natas_webpass/natas11 #` -> nope

`c /etc/natas_webpass/natas11 #` -> bingo

![](images/Pasted image 20210312172522.png)

We get the password for natas11