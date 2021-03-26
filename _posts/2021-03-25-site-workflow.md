---
title: site workflow
tags: intro workflow first
---


## Intro

I'd like to start off my first post with a simple writeup about my setup and workflow. Also i'd to mention [zacheller](https://zacheller.dev/) who inspired me to create this site and also wrote a post on his setup, i'd advise you to read his [writeup](https://zacheller.dev/jekyll-blog) too

I used jekyll which is a static site generator and also has builtin support for Github Pages, where this site is hosted. For more insite details you can view my github repo [here](https://github.com/saintbarber/saintbarber.github.io). I also use Obsidian as my personal note taking application, i created a symbolic link that points to the `_posts` folder in the jekyll root folder, meaning i can simply add a note in obsidian and it will also be posted on this site. 

## Getting Started

First i created an account on github and then created a repository named `<username>.github.io` this will automatically create the `<username>.github.io` gitpages website. Then i cloned the repo to my machine where i can work on it locally using VScode as my code editor. 

Next i chose a jekyll theme, i went with the [hacker blog theme](https://github.com/tocttou/hacker-blog), i cloned this repo and copied all the files to my own.

Now i used the command `jekyll serve --watch` which creates a local server on port `4000` and updates the site contents as you code.

## Creating a post

To create a post you need to create a file in the `_post` directory with the following format `YYYY-MM-DD-Website-Title`. The file must begin with the following lines:
```
___
title: Title goes here
___
```

And everything below will be renderd as kramdown the default markdown renderer for Jekyll.

## Workflow 
### Demo
Here is a small demo showing how i integrated obsidian with editing posts on my site.
(Sorry about the half-grey video, dont know how to fix it)

<div class="embed-container">
  <iframe
      src='images/demo.mp4'
      width="700"
      height="480"
      scrolling="no"
      frameborder="0"
      allowfullscreen="">
  </iframe>
</div>

### Obsidian 
As i said in the intro section i use obsidian as my personal note taking application. I created a symbolic link in my vaults root directory like so

`ln -s ~/saintbarber.github.io/_posts/ <vault-directory>/saintbarber.github.io`

![](images/Pasted%20image%2020210326000619.png)

Also, i have obsidian configured in a way that all pasted screenshots get automatically saved to a sub-folder called `images` (view obsidian settings) 

![](images/Pasted%20image%2020210326000959.png)

Then i created another symbolic link from the above `images` folder that points to another `images` folder in the root directory of my site

`ln -s ~/saintbarber.github.io/images/ <vault-directory>/saintbarber.github.io/images`

![](images/Pasted%20image%2020210326001237.png)

Now i can reference all my screenshots in obsidian.

_NOTE:_ Dont forget to add the images symbolic link to your `.gitignore` file.

![](images/Pasted%20image%2020210326013812.png)

### Jekyll

I changed some of the css code to match my preference and added my personal information to the `_config.yml` file. I also added `tags` to each post by editing the the `posts.html` file in the `_layouts` folder.

```liquid
{%raw%}
{%for tag in page.tags%}
	<div class="tag">{{ tag }}</div>
{%endfor%}
{%endraw%}
```

Now by adding the `tags` variable in the beginning of each post this will iterate through each tag and print it under the post title.

```kramdown
---
title: site workflow
tags: intro workflow first
---
```

I created a small script to create a post. 
```bash
#!/bin/bash

fullpath="/home/saintbarber/Notebooks/Hacking Notes/saintbarber.github.io/"

if (( $# < 1))
then
	echo "USAGE:  $0 <post name>"
	exit 1
fi

now=`date +%Y-%m-%d`
filename=`echo $now-$1 | tr ' ' '-'`
filename=$filename.md
filename=`echo $fullpath$filename`

touch "$filename"

echo "---" >> "$filename"
echo "title: $1" >> "$filename"
echo "tags: " >> "$filename"
echo "---" >> "$filename"
```

This script takes the post title as the first argument and creates the file with the correct format as mentioned above in my obsidian notes folder and then adds the boiler plate code for each post.

Once i finish writing my post i just commit and push to my repository.

From site root directory
```bash
git add .
git commit
git push
```

