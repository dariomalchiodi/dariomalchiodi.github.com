---
layout: post
title: "Securing a CentOS 7 Apache Web server through Let's Encrypt"
description: ""
category: System administration
tags: [Apache, CentOS 7, SSL]
---
{% include JB/setup %}
I recently needed to secure the apache installation in a virtual private
server running CentOS 7. I followed the instruction in this excellent
[tutorial](https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-centos-7)
by [DigitalOcean](https://www.digitalocean.com).

These instructions allow you to get up to the A rating for the
[Qualys SSL Server Test](https://www.ssllabs.com/ssltest/index.html).
To jump up to an A+ rating, the server must also redirect all HTTP
requests to their HTTPS equivalent. A straightforward way to do this is
through enabling HSTS (HTTP Strict Transport Security), adding the line

{% highlight console %}
Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains
{% endhighlight %}

to the `ssl.conf` configuration file in the `/etc/httpd/conf.d` directory.
Note that apache should be restarted in order for this change to take
effect:

{% highlight console %}
$ sudo apachectl restart
{% endhighlight %}

Happy SSL.
