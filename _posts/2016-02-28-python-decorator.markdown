---
layout: post
title:  "Python Decorator"
date:   2016-02-28 18:20:20 +0800
categories: python
tags:
- learn
- python
- decorator
---
<h1>1.Base Decorator</h1>
`code 1` is the same of `code 2`

[code 1]
{% highlight python %}
def deco1(func):
    print "before"
    func()
    print "after"

def func():
    print "in func"

if __name__ == '__main__':
    print "--------------"
    deco1(func)
    print "=============="

{% endhighlight %}

[code 2]
{% highlight python %}
def deco1(func):
    def _wrap():
        print "before"
        func()
        print "after"
    return _wrap

@deco1
def func():
    print "in func"

if __name__ == '__main__':
    print "--------------"
    func()
    print "=============="

{% endhighlight %}

theirs outputs are:

{% highlight html%}
--------------
before
in func
after
==============
{% endhighlight %}
<h1>2.Decorator with parameters</h1>
`code 3` is the same of `code 4`


[code 3]
{% highlight python %}
def deco1(receive_deco1_param):
    def _wrap(receive_function_name):
        def __wrap(receive_function_param):
            print "deco1's param is " + receive_deco1_param
            print "before"
            receive_function_name(receive_function_param)
            print "after"
        return __wrap
    return _wrap

def func(p):
    print "func's param is " + p

if __name__ == '__main__':
    print "--------------"
    deco1("deco1 param")(func)("func param")
    print "=============="
{% endhighlight %}

[code 4]
{% highlight python %}
def deco1(receive_deco1_param):
    def _wrap(receive_function_name):
        def __wrap(receive_function_param):
            print "deco1's param is " + receive_deco1_param
            print "before"
            receive_function_name(receive_function_param)
            print "after"
        return __wrap
    return _wrap

@deco1("deco1 param")
def func(p):
    print "func's param is " + p

if __name__ == '__main__':
    print "--------------"
    func("func param")
    print "=============="
{% endhighlight %}

theirs outputs are:

{% highlight html%}
--------------
deco1's param is deco1 param
before
func's param is func param
after
==============
{% endhighlight %}
<h1>3.The best practice of decorator maybe</h1>

[code 5]
{% highlight python %}
from functools import wraps

def deco1(receive_deco1_param):
    def _wrap(receive_function_name):
        @wraps(receive_function_name)
        def __wrap(receive_function_param):
            print "deco1's param is " + receive_deco1_param
            print "before"
            receive_function_name(receive_function_param)
            print "after"
        return __wrap
    return _wrap
	
@deco1("deco1 param")
def func(p):
    print "func's param is " + p

if __name__ == '__main__':
    print "--------------"
    func("func param")
    print func.__name__
    print "=============="
{% endhighlight %}
In `code 4`,if `print func.__name__` in `main`, output will be `__wrap`,but in `code 5`,the output is `func`.