---
layout: page
title: Hello World OpenStack!
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

- Keystone
- Heat
- cfn-tools
- git
- HA deployment

## Tips

{% highlight python linenos %}

if __name__ == '__main__':
    print 'Hello World OpenStack!'

{% endhighlight %}

----------

陈锐 Rui Chen @kiwik at sina chenrui.momo@gmail.com