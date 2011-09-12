---
layout: post
title: How to use infinity in your code
category: ruby
tags: ['ruby', 'tips']
---

You'll sometimes have to use the notion of infinity in your Ruby code. There isn't any way to directly use infinity in Ruby, at least none that I know of… But there's a tip to get it.

Let's say that you have a `next_row` method which has to return next line of an infinite array -- rows are added on the fly by another process.

In practice you'll certainly use `#loop` but we're going to use `#each` for the sake of demo.

{% highlight ruby %}
def each
  0.upto(1.0/0.0) { yield next_row }
end
{% endhighlight %}

A quick test in IRB tells use that `1.0 / 0.0` is interpreted as `Infinity`:

{% highlight ruby %}
>> 1.0/0.0
=> Infinity
{% endhighlight %}

but you can't use `Infinity` directly since it doesn't seems to be a class, an object or something else:

{% highlight ruby %}
>> Infinity
NameError: uninitialized constant Object::Infinity
{% endhighlight %}

Using this trick, our `#each` implementation will loop forever. It's definitively not the right solution to loop over elements but I thought it was interesting to know this little Ruby secret. 