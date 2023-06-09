---
layout: page
title: Sxmatch's Tech Life
tagline: Keep Yourself Up!
---
{% include JB/setup %}

## Ready blogs 

<ul class="posts">
  {% for post in site.posts %}
    <p><span>{{ post.date | date_to_string }}</span> » <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></p>
  {% endfor %}
</ul>

## Focuse list

- OpenStack
- Kubernetes
- Ceph
- AI
- Linux
- Python
- GO

## Tips

{% highlight python linenos %}

if __name__ == '__main__':
    print 'Hello World!'

{% endhighlight %}

-------
---

sxmatch Hao Wang @sxmatch at Twitter sxmatch1986@gmail.com
