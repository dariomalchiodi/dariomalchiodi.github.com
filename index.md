---
layout: page
title: MalchioLog
tagline: My digital memory
---
{% include JB/setup %}

<div class="containter">
  {% assign frontpost = site.posts[0] %}
  <div class="span12">
    <h1><a href="{{ frontpost.url }}">{{ frontpost.title }}</a><small> – {{ frontpost.date | date: "%Y/%m/%d" }}</small></h1>
    <p>
      In:
      {% if frontpost.categories.size > 0 %}
        {% for category in frontpost.categories %} <a href="/categories/{{ category }}" rel="tooltip" title="View posts in category &quot;{{ category }}&quot;">{{ category }}</a>  {% if forloop.last != true %} – {% endif %} {% endfor %}
      {% else %}
        no category
      {% endif %}
      Tags :
      {% if frontpost.tags.size > 0 %}
        {% for tag in frontpost.tags %} <a href="/tags/{{ tag }}" rel="tooltip" title="View posts tagged with &quot;{{ tag }}&quot;">{{ tag }}</a>  {% if forloop.last != true %} – {% endif %} {% endfor %}
      {% else %}
        no tags
      {% endif %}
    </p>
    <p>
      {{ frontpost.content | strip_html | truncatewords:45}}<br>
      <a href="{{ frontpost.url }}">Read more...</a>
    </p>
  </div>

  {% for post in site.posts offset: 1 limit: 11 %}
      
    <div class="span5">
      <h2><a href="{{ post.url }}">{{ post.title }}</a><small> – {{ post.date | date: "%Y/%m/%d" }}</small></h2>  
      <p>
        In:
        {% if post.categories.size > 0 %}
          {% for category in post.categories %} <a href="/categories/{{ category }}" rel="tooltip" title="View posts in category &quot;{{ category }}&quot;">{{ category }}</a>  {% if forloop.last != true %} – {% endif %} {% endfor %}
        {% else %}
          no category
        {% endif %}
        Tags :
        {% if post.tags.size > 0 %}
          {% for tag in post.tags %} <a href="/tags/{{ tag }}" rel="tooltip" title="View posts tagged with &quot;{{ tag }}&quot;">{{ tag }}</a>  {% if forloop.last != true %} – {% endif %} {% endfor %}
        {% else %}
          no tags
        {% endif %}
      </p>    
      <p>
        {{ post.content | strip_html | truncatewords:45}}<br>
        <a href="{{ post.url }}">Read more...</a>
        <a href="http://erjjones.github.com{{ post.url }}#disqus_thread" data-disqus-identifier="{{ post.url }}"></a>
      </p>

        
    </div>
  {% endfor %}


<!--   <div class="span2">
        <a href="{{ frontpost.url }}" >
            <img border="0" width="250" height="150" src="/img/posts/{{ post.image }}" alt="">
        </a>
      </div>
      <div class="span5">
    <h4><strong><a href="{{ frontpost.url }}">{{ frontpost.title }}</a></strong></h4>      
        <p>
          {{ frontpost.summary }}
        </p>
    <p>
          <i class="icon-calendar"></i> {{ frontpost.date | date: "%B %e, %Y" }}
          | <i class="icon-comment"></i> <a href="http://erjjones.github.com{{ post.url }}#disqus_thread" data-disqus-identifier="{{ post.url }}"></a>     
      | <i class="icon-tags"></i> Tags :{% for tag in frontpost.tags %} <a href="/tags/{{ tag }}" rel="tooltip" title="View posts tagged with &quot;{{ tag }}&quot;"><span class="label label-info">{{ tag }}</span></a>  {% if forloop.last != true %} {% endif %} {% endfor %}               
        </p>
        <p><a href="{{ post.url }}">Read more</a></p>
      </div> -->
</div>



<!-- Read [Jekyll Quick Start](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)

Complete usage and documentation available at: [Jekyll Bootstrap](http://jekyllbootstrap.com)

## Update Author Attributes

In `_config.yml` remember to specify your own data:
    
    title : My Blog =)
    
    author :
      name : Name Lastname
      email : blah@email.test
      github : username
      twitter : username

The theme should reference these variables whenever needed.

{% highlight ruby linenos %}
def foo
  puts 'foo'
end
{% endhighlight %}
    
## Sample Posts

This blog contains sample posts which help stage pages and blog data.
When you don't need the samples anymore just delete the `_posts/core-samples` folder.

    $ rm -rf _posts/core-samples

Here's a sample "posts list".

<ul class="posts">
  {% for post in site.posts limit 5 %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul> -->



