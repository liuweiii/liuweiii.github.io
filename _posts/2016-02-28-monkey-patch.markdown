---
layout: post
title:  "Monkey Patch [todo]"
date:   2016-02-28 18:20:20 +0800
categories: python
tags:
- learn
- python
- decorator
---

{% highlight python %}
def decorator(func):
  print "before"
  func()
  print "end"
{% endhighlight %}

