---
layout: post
tags: [intro, beginner, jekyll, tutorial]
description: How I Created this Jekyll Blog
---
{% include JB/setup %}

This is a quick technical overview of how I setup this blog and how easy it can be to
start your own!

### Jekyll ##

First off, this blog is based on [Jekyll](https://github.com/mojombo/jekyll) which in its own words *"is a
simple, blog aware, static site generator."* What this really means is that it
turns some simply formatted files in into full html pages for you. It does also
have a built in web server, but your not really meant to use this for a full
site deploy, only for testing.

This blog is based off [Jekyll-Bootstrap](http://jekyllbootstrap.com/) which
makes the process of setting up this blog even quicker!

To get started with Jekyll and Jekyll-Bootstrap, you will need [Ruby
Gems](http://rubygems.org/) installed which is essentially a package manager
for ruby programs.

If you want free hosting of your blog you will also need a
[github](http://github.com) account and
[git](http://git-scm.com/) installed.

### Getting Started ##

The first thing I did was go through the instructions on
the [Jekyll-Bootstrap](http://jekyllbootstrap.com/) page, which told me to:

* Create a github repository, called USERNAME.github.com.

Where you replace *USERNAME* your github username with. This is important as once you
create this repo github realises this is a Jekyll blog and turns this a
github pages site. Very nice!

Then:

	git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.com
	cd USERNAME.github.com
	git remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git
	git push origin master

Github will then automatically turn your site repo into a [Github
Pages](http://pages.github.com/) site. This may take a few minutes to update so
be patient, and it will take a few minutes to update every time you push your
changes.

You can actually view all the source from this blog
[here](http://github.com/takac/takac.github.com).

Now there are some very good resources already on getting your site up and
running, and adding new posts so I recommend checking out the
[Jekyll](https://github.com/mojombo/jekyll) documentation before going any
further.

### Running Jekyll and Customising ##

Now you will probably want to test your site locally before pushing you changes
to the live site. This can be accomplished very easily using Jekyll.
    
	jekyll --server --auto

Make sure you are in the root of your repository when you run this. This will
start Jekyll running a web server on
[http://localhost:4000/](http://localhost:4000/). As we have specified the
`--auto` option to Jekyll, it will automagically compile any changes to our site
and rehost them. 

If you now browse to [http://localhost:4000/](http://localhost:4000/) you see a
very generic looking blog, with a nice intro to using Jekyll-Bootstrap and Jekyll.
However it doesn't have a nice blog name or any indication of whos blog it is!
Lets change that. 

On you file system you will need to browse to your repository and open up 
\_config.yml in your favourite text editor. Mine is Vim, and I will certianly
discuss that later... 

	title : Blog Title
	tagline: computers. commandline. java. blog.
	author :
	  name : Tom Cammann
	  email : your.email@here.com
	  github : takac
	  twitter : tea_sea
	  feedburner : feedname
	
Go ahead and change this to you own information and blog information. You can
also setup [Google Analytics](http://www.google.co.uk/analytics/),
[Feedburner](http://feedburner.com) and [Disqus](http://www.disqus.com) with
your Jekyll blog, however this is more advanced stuff.

Once you save your details in there, you should be able to browse to
localhost:4000 again and view the updated changes.

Now we have setup the config correctly we can modify our landing page, or index
page, which is set to `index.md`. Again open up your text editor and start
changing! The format of the files your Jekyll blog will use is based on
[Markdown](http://daringfireball.net/projects/markdown/) which *allows you to
write using an easy-to-read, easy-to-write plain text format*, which is then
converted into HTML by Jekyll. Again after you have made some changes to 
`index.md` and you have saved your changes you can view the rendered page by
browsing to [localhost:4000](http://localhost:4000).

### First Post ##

Now you've customised your own front page it seems like a good time to have a
go a your first blog post. Each post needs to go in the `_post` directory. The
filename is also very important. The filename must follow the convention: 
`yyyy-mm-dd-title-of-post.markup`, eg. this file is called
`2013-01-11-how-this-all-got-started.md`. The markup as the file extension
specifies what markup language this file will be written in. I like markdown,
and I have used the markdown extension `.md` so I can use it.

Create a new file: `_posts/2013-01-11-my-first-post.md`

You must start your post with a `yaml` header. 

	---
	layout: post
	description: How I Created this Jekyll Blog
	tags: [intro, beginner, jekyll, tutorial]
	---

The layout key is the most important line. It specifies what layout template to
use when building the HTML page. All the layouts available for your site are in
the `_layouts` directory. This means you can also customise any of your layouts
at anytime and it will update the other posts using that layout.

The tags and description lines just specify some extra metadata for your site.
They aren't necessary but they add helpful metadata which really helps for SEO.

After the `yaml` header in the Jekyll-Bootstrap posts, there is a strange
looking line:

	{% include JB/setup %}

This line is in the [Liquid Markup](http://liquidmarkup.org/) syntax. It is
another markup language used within any of the pages you create that are
processed by Jekyll. This line tells Liquid to directly include the contents of
the file `JB/setup`. This file can also be in the same format, using Liquid
tags {% raw %} `{% ... %}` {% endraw %}.

The rest of the content of your must be in your specified markup language, but
you can include Liquid markup at any point in the document too. Lets create a
header for our post, and some text underneath.

	## My First Header ##
	The ## signals a H2 style header, if you used one # it would be an H1 style
	header, and if you used 3#s it would be an H3 style header. This text will
	come out as a standard text body.


Great now we have the started our blog post, we should save and see what this
renders too. If you haven't already, lets start Jekyll and go
[http://localhost:4000/](http://localhost:4000/). Jekyll can be started by
running:

	jekyll --server --auto

In the root directory of your blog, ie in the same directory as the `_posts` and
`_layouts` directories.

We should now see a nice clean web render of your really simple Markdown post.
Great! 

### Additional Resources ##

Hopefully this helped get you started blogging with Jekyll. There are lots of
other great resources out there to help you get started. Make sure you checkout
the [Jekyll documentation]() and the
[Jekyll-Bootstrap](http://jekyllbootstrap.com/lessons/jekyll-introduction.html)
documentation, they were both helpful for me when starting this blog. 

Another invaluable resource is the sites that other developers have created
using Jekyll. The source code for many of the sites that use Jekyll are listed
[here](https://github.com/mojombo/jekyll/wiki/Sites). Go check them out and get
inspired!

Tom
