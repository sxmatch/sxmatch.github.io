---
layout: page
title: OpenStack I'm here!
tagline: keep simple
---
{% include JB/setup %}

## Ready blogs 

<ul class="posts">
  {% for post in site.posts %}
    <p><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></p>
  {% endfor %}
</ul>

## Doing list

- Cinder
- Zaqar
- Ceph
- git

## Tips

{% highlight python linenos %}

if __name__ == '__main__':
    print 'Hello World OpenStack!'

{% endhighlight %}

----------

sxmatch Wang Hao @sxmatch at sina sxmatch1986@gmail.com