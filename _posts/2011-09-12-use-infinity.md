---
title: How to use infinity in your code
category: Ruby
tags: ['best practices', 'tips']
header:
  overlay_image: /assets/images/headers/ruby.png
  overlay_color: "#4D4D4D"
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

To use `Infinity` directly without this trick you can use:

{% highlight ruby %}
>> Float::INFINITY
=> Infinity
{% endhighlight %}

Now our `#each` implementation will loop forever. It's definitively not the right solution to loop over elements but I thought it was interesting to know this little Ruby secret.
