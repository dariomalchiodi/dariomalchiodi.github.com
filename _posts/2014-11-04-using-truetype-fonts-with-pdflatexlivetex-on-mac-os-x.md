---
layout: post
title: "Using TrueType fonts with pdflatex/livetex on Mac OS X"
description: ""
category: 
tags: []
---
{% include JB/setup %}
My University recently established a set of typographic conventions to be used
when producing official documents such as presentations. A Powerpoint template
was provided, but that was almost useless to me because I'm more a LaTeX guy.
Aiming at building a beamer template compliant with the above conventions, I
had to learn how to typeset LaTeX documents using the Trebuchet font in Mac OS
X Mavericks equipped with pdflatex and livetex 2014. The steps I took should be
independent

- of the specific operating system,
- of the livetex release, and
- of the chosen font, as soon as it belongs to the TrueType family.

First of all, it is advisable to create a temporary directory within which
building the font files:

{% highlight console %}
~$ mkdir tmp
~$ cd tmp
{% endhighlight %}

The following step consists in finding the TrueType descriptions of the chosen
font. This is easy for Mac users, as the corresponding files are shipped with
the OS in the `/Library/Fonts` directory. For a given font there are typically
four descriptions, respectively for the standard form and the bold, italic and
bold+italic variants. The corresponding files have a name made up of the font
name, the considered variant and a `ttf` extension. Such files should be copied
to our temporary directory, taking care of removing spaces in file names and
possibily adhering to the convention of adding the suffix "b", "i", and "z" for
the bold, italic, and bold+italic variants:

{% highlight console %}
~/tmp$ cp /Library/Fonts/Trebuchet*ttf .
~/tmp$ mv Trebuchet\ MS.ttf trebuchet.ttf
~/tmp$ mv Trebuchet\ MS\ Bold.ttf trebuchetb.ttf
~/tmp$ mv Trebuchet\ MS\ Italic.ttf trebucheti.ttf
~/tmp$ mv Trebuchet\ MS\ Bold\ Italic.ttf trebuchetz.ttf
{% endhighlight %} 

Next, we need to find out where three specific tex-aware directory are within
the file system. In order to find out the exact pathnames, which are dependent
on the specific texlive release, we can use the `kpsewhich` utility and set up
three environment variables to be used henceforth:

{% highlight console %}
~/tmp$ export TEXMFDIST=`kpsewhich -var-value=TEXMFDIST`
~/tmp$ export TEXMFHOME=`kpsewhich -var-value=TEXMFHOME`
~/tmp$ export TEXMFSYSVAR=`kpsewhich -var-value=TEXMFSYSVAR`
{% endhighlight %}

In order to get the metric files for the font to be installed, we need to build
some intermediate files, encoded in the adobe font metric format, through the
`ttf2afm` utility:

{% highlight console %}
~/tmp$ ttf2afm -e $TEXMFDIST/fonts/enc/ttf2pk/base/T1-WGL4.enc -o ectrebuchet.afm trebuchet.ttf
~/tmp$ ttf2afm -e $TEXMFDIST/fonts/enc/ttf2pk/base/T1-WGL4.enc -o ectrebucheti.afm trebucheti.ttf
~/tmp$ ttf2afm -e $TEXMFDIST/fonts/enc/ttf2pk/base/T1-WGL4.enc -o ectrebuchetb.afm trebuchetb.ttf
~/tmp$ ttf2afm -e $TEXMFDIST/fonts/enc/ttf2pk/base/T1-WGL4.enc -o ectrebuchetz.afm trebuchetz.ttf
{% endhighlight %}

Note that the `T1-WGL4.enc` encoding table could contain definitions for glyps
not included in the font we are installing: in such case, one or more warnings
(such as ``"glyph `ffl' not found"``) show up in stderr.

The .afm files can now be used in order to produce the font metrics which
texlive expects (`...` means a skipped verbose output):

{% highlight console %}
~/tmp$ afm2tfm ectrebuchet.afm -T $TEXMFDIST/fonts/enc/ttf2pk/base/T1-WGL4.enc ectrebuchet.tfm
...
~/tmp$ afm2tfm ectrebucheti.afm -T $TEXMFDIST/fonts/enc/ttf2pk/base/T1-WGL4.enc ectrebucheti.tfm
...
~/tmp$ afm2tfm ectrebuchetb.afm -T $TEXMFDIST/fonts/enc/ttf2pk/base/T1-WGL4.enc ectrebuchetb.tfm
...
~/tmp$ afm2tfm ectrebuchetz.afm -T $TEXMFDIST/fonts/enc/ttf2pk/base/T1-WGL4.enc ectrebuchetz.tfm
{% endhighlight %}

Now that Trutype and font metric files have been generated, we also need to
create directories to hold them:

{% highlight console %}
sudo mkdir -p $TEXMFDIST/fonts/truetype/ms
sudo mkdir -p $TEXMFDIST/fonts/tfm/ms
{% endhighlight %}

We are now ready to place all built files where texlive expects them to be.
Note that `$TEXMFDIST` will typically point to a system directory, and in this
case you will need to get access to it (likely through `sudo`):

{% highlight console %}
~/tmp$ cp *ttf $TEXMFDIST/fonts/truetype/ms/
~/tmp$ cp *tfm $TEXMFDIST/texlive/2014/texmf-dist/fonts/tfm/ms/
{% endhighlight %}

To let texlive aware of the new font it is sufficient to add the following
lines at the end of `$TEXMFSYSVAR/fonts/map/pdftex/updmap/pdftex.map` (again,
to be likely accessed through `sudo`):

{% highlight console %}
ectrebuchet Trebuchet " T1Encoding ReEncodeFont " <trebuchet.ttf T1-WGL4.enc
ectrebucheti Trebuchet-Italic " T1Encoding ReEncodeFont " <trebucheti.ttf T1-WGL4.enc
ectrebuchetb Trebuchet-Bold " T1Encoding ReEncodeFont " <trebuchetb.ttf T1-WGL4.enc
ectrebuchetz Trebuchet-BoldItalic " T1Encoding ReEncodeFont " <trebuchetz.ttf T1-WGL4.enc
{% endhighlight %}

The last step consists in invoking the `mktexlsr` utility, which builds a
database of files found in specific texlive directories:

{% highlight console %}
sudo mktexlsr
...
{% endhighlight %}

To access the new fonts from LaTeX, a new style file has to be created.
Although other options are possible, this file typically has the same name of
the installed font, and is located in the `tex/latex` directory under the user
texmf tree: thus edit `$TEXMFHOME/tex/latex/trebuchet.sty` as follows: 

{% highlight tex %}
\DeclareFontFamily{T1}{trebuchet}{}%
\DeclareFontShape{T1}{trebuchet}{b}{n}{<->ectrebuchetb}{}%
\DeclareFontShape{T1}{trebuchet}{b}{it}{<-> ectrenuchetz}{}%
%% bold extended (bx) are simply bold
\DeclareFontShape{T1}{trebuchet}{bx}{n}{<->ssub * trebuchet/b/n}{}%
\DeclareFontShape{T1}{trebuchet}{bx}{it}{<->ssub * trebuchet/b/it}{}%
\DeclareFontShape{T1}{trebuchet}{m}{n}{<-> ectrebuchet}{}%
\DeclareFontShape{T1}{trebuchet}{m}{it}{<-> ectrebucheti}{}%
\usepackage[T1]{fontenc}%
\renewcommand{\rmdefault}{trebuchet}%
\renewcommand{\sfdefault}{trebuchet}%
{% endhighlight %}

To test that the installation is fine create a simple LaTeX document using
the `trebuchet` package, such as the following one:

{% highlight latex %}
\documentclass{article}
\usepackage{trebuchet}
\begin{document}
Test
\end{document}
{% endhighlight %}

and `pdflatex` it to check that no errors rise up and that the produced PDF
actually contains the new font (either visually or through an utility such as
`pdffonts`). Finally, remember to clean up things by simply removing the
temporary directory:

{% highlight console %}
~/tmp$ cd ..
~/tmp$ rm -rf tmp
{% endhighlight %}

