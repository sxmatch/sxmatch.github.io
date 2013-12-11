---
layout: page
title: Hello World!
tagline: Supporting tagline
---
{% include JB/setup %}

## 最近在做的事

    GAE
    Github blog
    rpm
    Jekyll

    
## Ready
我已经完成的blogs 

{% highlight bash linenos %}

    $ find . -name "*.tmp" | xargs rm -rf

{% endhighlight %}

```shell
def yourfunction():
     print "Hello World!"
```

My blog list.

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## To-Do
My doing list.

<ul class="posts">
  {% for draft in site.drafts %}
    <li> &raquo; <span>{{ draft.title }}</span></li>
  {% endfor %}
</ul>

## From

Read [Jekyll Quick Start](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)

Complete usage and documentation available at: [Jekyll Bootstrap](http://jekyllbootstrap.com)

