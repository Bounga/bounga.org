---
title: Encapsulate a code block in an object
category: Ruby
tags: ['tips']
header:
  overlay_image: /assets/images/headers/ruby.jpg
  overlay_filter: 0.2
---

[Ruby](http://www.ruby-lang.org) is a very dynamic language which allows you to do things like encapsulate a piece of code in an object. This is a really powerful capability you can use to write flexible and reusable code. This concept is based on *Proc* objects.

This post will show you how you can use *Proc*s to simplify your code, avoid duplication and makes it more dynamic.

How to create a Proc?
=====================

As easy as pie!

{% highlight irb %}
>> proc = Proc.new { |x| puts "Parameter is #{x}" }
=> #<Proc:0x000000010140a720@(irb):1>
>> proc.call('test')
Parameter is test
=> nil
{% endhighlight %}

We create a new instance of the *Proc* class using the *new* method. Initializer method needs a block that will define its behavior on each *Proc#call* method call. In this example, our block only prints its parameter.


On-the-fly Procs
----------------

*Proc* objects can be created on-the-fly on a method call:

{% highlight ruby %}
def method_using_a_block(x, &block)
	puts block.class
	x.times { |i| block.call([i, i*i]) }
end
{% endhighlight %}

Here is what happens when you call the previous method:

{% highlight irb %}
>> method_using_a_block(4) { |n, s| puts "Square of #{n} is #{s}" }
Proc
Square of 0 is 0
Square of 1 is 1
Square of 2 is 4
Square of 3 is 9
=> 4
{% endhighlight %}

As you can see, the method takes two parameters — an integer and a block. The first parameter represents iteration count — How many times will I call the given block? The second parameter is the block itself, it represents what the method has to do with the calculated values. In this case it doesn't do much, it only prints values.

Before printing values we're printing block class just to show it's really a *Proc* object which was build on-the-fly. Then the block is called *x* times. On each iteration the block is fed with current *x* value and its square. The block doesn't calculate these values, it only uses them.

Tired of typing? Here's a shortcut
----------------------------------

{% highlight ruby %}
block.call([i, i*i])
{% endhighlight %}

can be written

{% highlight ruby %}
block[i, i*i]
{% endhighlight %}

Yes but I already have a Proc object!
=====================================

It's also possible to use an existing *Proc* object rather than using an inline block. The only thing you have to remember is to prefixed your variable by a &:

{% highlight irb %}
>> proc = Proc.new { |n| puts n }
=> #<Proc:0x00000001014b03f0@(irb):1>

>> 3.times(&proc)
0
1
2
=> 3
{% endhighlight %}

That's fun but any real use cases?
======================================

For sure! Let's say you're writing a Rails app and you need to be able to sort your blog posts by title, author, descending id and state (published / draft). Using *Proc* could really save you some time and lines of code.

Even if I know that sorting should be done at DB level for performance reasons, one of the possible implementation showing *Proc* usage could be:

{% highlight ruby %}
def sort
	order = params[:order] || "id"
	proc = case order
	when "author" then lambda { |p| p.user.name.downcase }
	when "status", "title" then lambda { |p| p.send(order) }
	when "id" then lambda { |p| -p.id }
	end

	@posts = Post.all.sort_by &proc
end
{% endhighlight %}

First we're defining an *order* variable to store the sorting parameter that we've retrieved from POST or GET parameters. If there's no *order* parameter we'll use *id* by default.

We then populate the *proc* variable with a *lambda* according to the requested order — *lambda* is in fact a specialized *Proc* object that checks its parameters:
- user wants to sort by author: we'll use downcased *name* attribute of the *user* associated to the post
- user wants to sort based on the post *status* or the post *title*: we'll use the attribute without any further processing. We're using an `Object#send` call to avoid duplication and conditions. The corresponding method is called dynamically
- user wants to sort based on descending *id*: we just have to call the corresponding method but prefixed by a minus to sort in reverse order

Now we can fetch all posts using ActiveRecord and sort the resulting collection using *Enumerable#sort_by* method which takes a block (our proc) as its parameter to know how to sort the array.

Nothing spectacular but I really think it can be useful in many cases. It leads to a shorter and more maintainable code. When you have a lot of duplication with small differences you really should think of *Proc* to keep your code DRY.

For example I heavily used *Proc* for one of my command-line software [Renamer](https://rubygems.org/gems/Renamer). It can be a good study case for a better understanding of *Proc*s.

Resources
=========

- [Proc class](http://apidock.com/ruby/Proc)
- [Kernel#lambda](http://apidock.com/ruby/Kernel/lambda)
- [Renamer source code](https://bitbucket.org/Bounga/renamer/src/)
