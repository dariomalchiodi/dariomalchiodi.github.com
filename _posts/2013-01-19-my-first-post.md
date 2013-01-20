---
layout: post
title: "Let me introduce... my blog"
description: ""
categories: [General]
tags: [Blog, Jekyll, Bootstrap, Markdown, Liquid]
---
{% include JB/setup %}
I decided to set up this blog mostly with the aim of writing down install
instructions as neat as possible for the software I use, so I (hopefully)
can save time when setup a new computer. So, why don't
describe the technology behind the blog itself as a first post?

Quick and dirty: the blog is a set of [HTML](http://en.wikipedia.org/wiki/HTML)
pages hosted on [github](http://github.com) statically generated through
[Jekyll](http://jekyllrb.com) using the twitter theme for the
[Jekyll-Bootstrap](http://jekyllbootstrap.com) framework, with comments support
provided by [Disqus](http://diqus.com).

Let me explain all this with a bit more of detail. The site contents are not
dynamically generated or read from a database: the actual HTML code your browser
is rendering is stored somewhere at github. The other endpoint is a set of text
files holding the blog contents, encoded using the markdown language. These files
are sent from my computer to the github server, where a text processor called
Jekyll transforms them into HTML code.

Although the not so low number of involved software, the required installation is
relatively simple, and basically follows the workflow described in the
[Jekyll-Bootstrap site](http://jekyllbootstrap.com) and in a specific [post from
Eric Jones](http://erjjones.github.com/blog/How-I-built-my-blog-in-one-day).

1. Set up a github account, if you don't already have one. I assume you know what
   a version control systems is, as well a working knowledge of the basic
   features of git. If this is not the case, google for a git tutorial.
2. Once logged into github, create a new repository named
   `YOURUSERNAME.github.com`, where `YOURUSERNAME` should be replaced by your
   actual github user name. Dont'add a README.md file or you'll run into a
   conflict later on.
3. Position yourself in a suitable place of your file system, and clone locally
   Jekyll-Bootstrap in a directory named `YOURUSERNAME.github.com`, then `cd` in
   that directory:

        $ git clone https://github.com/plusjade/jekyll-bootstrap.git YOURUSERNAME.github.com
		$ cd YOURUSERNAME.github.com

4. Associate the downloaded software with your github repository and push it:

		$ git remote set-url origin git@github.com:YOURUSERNAME/YOURUSERNAME.github.com.git
		$ git push origin master

The essential is done, and if you point your browser to `http://YOURUSERNAME.github.com`
you should see a template Web site. Note that generating the online web site requires
time, up to some minutes, so you could not see your site immediately.

Actually it would be better to have a development site upon which experimenting,
otherwise you'll have to make modifications directly on the publicly available site.
Thus, unless you really like to live dangerously, you'd better install a local copy of 
ekyll. This requires a working Ruby environment, which is better to update to a recent
version before proceeding to the installation:

    $ sudo gem update --system
    $ sudo gem install jekyll

When these two steps complete (and this could require some time), you can run a
Jekyll server:

    $ jekyll --server

This will read the markdown description of the template site delivered with Jekyll-Bootstrap,
generate the corresponding HTML and serving it on `localhost:4000`. The nice thing is that
the server is aware of the markdown files, so that any update on them will force the server
to generate again the HTML. Also in this case, the Web site generation requires some time,
although sensibly lower than required when publishing online. Roughly speaking, the new
content will usually show up when the page is reloaded once or twice.

Once the system is in place, you need to know how to write content. This requires to learn

1. the markdown format, describing the blog content using exclusively the keyboard characters;
2. the liquid template engine, allowing the blog description to be unaware of specific
   details such as the contents of a given post or tags and categories associated to it.

You can google tutorials or cheat sheets for the syntax of both tools. Just to give a first
reference, you can read the [Markdown tutorial by John Gruber](http://daringfireball.net/projects/markdown/) and the [Liquid for Designers](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) section of the Shopify site
on github.

Content can be added either as a new post or a new page: the former is the usual
content of a blog, while the latter represents additional content such as a Bio.
The syntax for adding both elements is similar:

	$ rake post title="TITLE"
	$ rake page name="NAME"

The text within double quotes represents the title of the post in the first example,
and the name of the markdown file containing the page description in the second one.
You'll find it in the home directory, unless the specified name is a pathname. Also
the first example creates a new file, which resides in the `_posts` directory and
which you will have to edit with the actual post content.

Once you are satisfied with your blog, publishing it online is simply a matter of
committing and pushing your repository:

	$ git add .
	$ git commit -m "Add new content"
	$ git push origin master

Another operation worth considering is that of customizing your blog. Also in this case
you can refer to the [Jekyll-Bootstrap site](http://jekyllbootstrap.com/usage/blog-configuration.html)
and to [Eric Jones' post](http://erjjones.github.com/blog/How-I-built-my-blog-in-one-day).
The only things I did not find there are the following ones.

1. The CSS provided with the downloaded version of Jekyll-Bootstrap is not responsive,
   that is it refers to a fixed layout requiring to horizontally scroll the browser
   contents if its window is too narrow. This is particularly annoying when browsing
   through a device with limited screen size, such as a smartphone. The solution to this
   problem is that of replacing the file `assets/themes/twitter/bootstrap/css/bootstrap.min.css`
   with a customized version built and downloaded from the
   [Bootstrap site](http://twitter.github.com/bootstrap/customize.html) (be sure to select
   the options in the *Responsive* section).
2. Source code highlighting needs to be explicitly turned on. This requires to download a
   CSS file (such as for instance [this one](https://raw.github.com/mojombo/tpw/master/css/syntax.css)),
   to be copied in the `assets/themes/twitter/css` directory and referenced in
   `_includes/themes/twitter/default.html` through a `link` tag:

{% highlight html %}
<link href="{% raw %}{{ ASSET_PATH }}{% endraw %}/css/syntax.css" rel="stylesheet" type="text/css" media="all">
{% endhighlight %}
