---
layout: post
title:  "Some awesome Ruby code"
date:   2015-10-25 6:44:00
categories: ruby
subcategory: intermediate
---

## Store method in variable
(Ruby functional programming properties using `method`)
If you want to store a method rather than the result of calling a method
or just the message you'd send to invoke it, you need to use the method method on the owning object.
So for example.

{% highlight ruby %}
"hello".method(:+)
{% endhighlight %}

will return the + method of the object "hello", so that if you call it with the argument " world",
you'll get "hello world".

{% highlight ruby %}
helloplus = "hello".method(:+)
helloplus.call " world" # => "hello world"
{% endhighlight %}


## Everything return something
In ruby every statement and block of code return some value even  <code>class</code></p>
{% highlight ruby %}
a = class A
  def initialize
    @a = 11
    @@a = 22
    a = 33
  end

  @a = 1
  @@a = 2
  a = 3
end

A.instance_variable_get(:@a) # => 1
A.class_variable_get(:@@a) # => 2
a # => 3
A.new.instance_variable_get(:@a) # => 11
A.class_variable_get(:@@a) # => 22
A.new.method(:initialize).call # => 33 , basically we tricked Ruby into calling init
{% endhighlight %}
