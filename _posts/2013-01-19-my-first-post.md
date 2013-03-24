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
pages, styled through [Bootstrap](http://twitter.github.com/bootstrap/), hosted
on [github](http://github.com) statically generated through
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

I'll assume you have a working knowledge of version control systems, you are
comfortable with git and you have a github account. If not, google for a git
tutorial (for instance [this one](http://learn.github.com/p/intro.html)),
[play around for a while](http://try.github.com), and [set
up](http://github.com/signup) a github account. I'll also assume you have an
updated version of Ruby on your system (execute `sudo gem update --system` to
check for updates on your Ruby installation)

1. Once logged into github, create a new repository named
   `YOURUSERNAME.github.com`, where `YOURUSERNAME` should be replaced by your
   actual github user name (in this step as well as in all subsequent occurrences
   of `YOURUSERNAME`. Don't let github add a README.md file, or you'll run
   into a conflict later on.
2. Position yourself in a suitable place of your file system, and clone locally
   Jekyll-Bootstrap in a directory named `YOURUSERNAME.github.com`, then `cd` in
   that directory:

       $ git clone https://github.com/plusjade/jekyll-bootstrap.git YOURUSERNAME.github.com
       $ cd YOURUSERNAME.github.com

3. Associate the downloaded software with your github repository and push it:

       $ git remote set-url origin git@github.com:YOURUSERNAME/YOURUSERNAME.github.com.git
       $ git push origin master

At this point, all the essential software is installed: point your browser to
`http://YOURUSERNAME.github.com` and you should see the Jekyll-Bootstrap template
Web site. Note that generating the online web site could require up to some minutes,
so don't worry you don't see your site immediately.

Unless you really like to live dangerously, it is wise to work on a development site
instead of directly modify the publicly available site. Thus, it is strongly recommended
that you also install a local copy of Jekyll, which installs as a Ruby gem:

{% highlight sh %}
$ sudo gem install jekyll
{% endhighlight %}

When this step completes (and this could require some time), you can run a
Jekyll server:

{% highlight sh %}
$ jekyll --server
{% endhighlight %}

It is important that the latter operation is executed within the
`YOURUSERNAME.github.com` directory: this will allow Jekyll-Bootstrap to read its contents
and convert them into a Web site locally available at the address `localhost:4000`.
The conversion works as follows: all files whose name does not start with an underscore
are read and trasformed using:

* a [Markdown](http://daringfireball.net/projects/markdown/) parser in order to
  handle the static part of the site contents: this allows blog entries to be written
  even without knowing anything about HTML, but simply rendering structure and formatting
  through a simple [textual-based
  syntax](http://daringfireball.net/projects/markdown/syntax);
* the [Liquid templating engine](http://liquidmarkup.org/) with the aim of rendering the
  dynamic part of the site and factoring common and repeated parts.

All files whose name begins with an underscore is left untouched. Such files contain
configuration options, assets such as CSS and JS, templates and so on. In particular,
under `_site` you'll find a copy of the generated Web site.

The Jekyll server is aware of file modification, so the locally served Web site is always
updated. Also in this case, the Web site generation requires some time,
although sensibly lower than required when publishing online. Roughly speaking, the new
content will usually show up when the page is reloaded once or twice.

You can google tutorials or cheat sheets for the syntax of both tools. Just to give a
first reference, you can read the [Markdown tutorial by John
Gruber](http://daringfireball.net/projects/markdown/) and the [Liquid for
Designers](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) section of the
Shopify site on github.

Content can be added either as a new post or a new page: the former is the usual
atomic content of a blog, while the latter represents a site page, such as a Bio, an
archive of past posts and so on. The syntax for adding both elements is similar, and
is based on `rake`, a building tool analogous to `make` but relying on Ruby:

{% highlight sh %}
$ rake post title="TITLE"
$ rake page name="NAME"
{% endhighlight %}

The text within double quotes represents the title of the post in the first example,
and the name of the markdown file containing the page description in the second one.
Created pages will reside in the home directory, unless the specified name is a
pathname, while posts are placed in the `_posts` directory, in files named aggregating
the current date and the specified title. It is precisely this file which you'll have
to edit, possibly using Markdown syntax, in order to insert the actual post content.

Once you are satisfied with your blog, publishing it online is simply a matter of
committing and pushing your repository:

{% highlight sh %}
$ git add .
$ git commit -m "Added new content"
$ git push origin master
{% endhighlight %}

Another operation worth considering is that of customizing your blog: this will likely
require to write general information and setting into `_config.yml` and to modify files
in the `_layouts` and `_includes` directories. The [Jekyll-Bootstrap
site](http://jekyllbootstrap.com/usage/blog-configuration.html) and the [Eric Jones'
blog](http://erjjones.github.com/blog/How-I-built-my-blog-in-one-day) represent
valuable resources. The only things I did not find there are the following ones.

1. The CSS provided with the downloaded version of Jekyll-Bootstrap is not responsive,
   that is it refers to a fixed layout requiring to horizontally scroll the browser
   contents if its window is too narrow. This is particularly annoying when browsing
   through a device with limited screen size, such as a smartphone. The solution to this
   problem is that of replacing the file
   `assets/themes/twitter/bootstrap/css/bootstrap.min.css` with a customized version
   built and downloaded from the [Bootstrap
   site](http://twitter.github.com/bootstrap/customize.html) (be sure to select
   the options in the *Responsive* section).
2. Source code highlighting needs to be explicitly turned on. This requires to install
   [pygments](http://pygments.org)

       $ sudo easy_install Pygments

    (using one of the methods in section *Pygments* of the [Jekyll
    site](https://github.com/mojombo/jekyll/wiki/Install) if `easy_instal` is not
    available), and to create a CSS file containing the source code formatting rules

        $ pygmentize -S default -f html > assets/css/pygments.css

    This file has to be copied in the `assets/themes/twitter/css` directory and
    referenced in `_includes/themes/twitter/default.html` through a `link` tag:

        <link href="{% raw %}{{ ASSET_PATH }}{% endraw %}/css/syntax.css" rel="stylesheet" type="text/css" media="all">

3. This last point is not explicitly connected with Jekyll-Bootstrap, but it's a
   part of what I wanted as default facility in my blog, that is proper mathematical
   formatting. My choice is [MathJax](http://www.mathjax.org), which does not
   (necessarily) require local installation: it's just a matter of remotely including
   the following javascript code into `_includes/themes/twitterdefault.html`:

       <script type="text/javascript" src="https://c328740.ssl.cf1.rackcdn.com/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

    which is ugly, but is the only safe way currently provided to avoid
    man-in-the-middle attacks. But in the end, this will allow your posts to
    have nice formulas such as

    $$ \frac{1}{\sqrt{2 \pi} \sigma} \int_{-\infty}}^{+\infty} \mathrm e^{\frac{(x - \mu)^2}{2 \sigma^2}} \mathrm d x = 1 $$

    Remeber, though,
    that inline equations should escape backslash in delimiters, thus using `\\(` and
    `\\)`instead of `\(` and `\)`.
