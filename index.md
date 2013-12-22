---
layout: page
title: Hello World!
tagline: regexisart
---
{% include JB/setup %}

## Ready blogs 

<ul class="posts">
  {% for post in site.posts %}
    <p><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></p>
  {% endfor %}
</ul>

## Doing list

- Github
- blog
- win-sshfs
- Vagrant
- DevStack

## Tips

{% highlight bash linenos %}

find . -name "*.tmp" | xargs rm -rf 

{% endhighlight %}

----------

陈锐 ruichen @kiwik chenrui.momo@gmail.com