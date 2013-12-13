---
layout: page
title: Hello World!
tagline: regexisart
---
{% include JB/setup %}

## Ready
我已经完成的blogs 

<ul class="posts">
  {% for post in site.posts %}
    <p><li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li></p>
  {% endfor %}
</ul>

## 最近在做的事
```
GAE
Github
blog
keystone
Jekyll
```
## Tips

```ksh linenos
$ find . -name "*.tmp" | xargs rm -rf
```endhighlight

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

