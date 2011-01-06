---
layout: post
title: Encapsulate a code block in an object
category: ruby
tags: ['ruby', 'tips']
---

Ruby is a very dynamic language and it allows you to do things like encapsulate a piece of code in an object. This is a really powerful trick since your code becomes usable in many places regarding the context. To sum this up we can compare this to Ruby blocks but embed in an object.

To use this you need to create a `Proc` object. We'll try to see in this post how we can use `Proc` to simplify our code and makes it more dynamic.
    
How to create a Proc?
=======================

Easy as a pie!

{{ highlight ruby}}
proc = Proc.new { |x| puts "Parameter is #{x}" }
proc.call('test') # => Parameter is test
{{ endhighlight }}

So we've created a new instance of the *Proc* class using the `new` method. This `new` method needs block wich will be use on each `Proc#call` method call. In this example, our block is waiting for a parameter that it will use after.

You can also create `Proc` objects on the fly in method calls:

{{ highlight ruby }}
def method_using_a_block(x, &block)
	puts block.class
	x.times { |i| block.call[i, i*i] }
end
{{ endhighlight }}

Then we can call our method:

{{ highlight irb }}
method_using_a_block(4) { |n, s| puts "Square of #{n} is #{s}" }

# => Proc
# => Square of 0 is 0
# => Square of 1 is 1
# => Square of 2 is 4
# => Square of 3 is 9
{{ endhighlight }}

We've defined a method which takes 2 parameters — an integer and a block. This method starts by printing our block class to be sure we're creating a `Proc` object. This block will be called `x` times and will be feed with current `x` value and it's square. In our case the code we use for the block only have a simple purpose, displaying a sentence explaining what is the square of a given number.

Note that 

{{ highlight ruby }}
block.call[i, i*i]
{{ endhighlight }}

could be shorten to

{{ highlight ruby }}
block[i, i*i]
{{ endhighlight }}

Yes but I already have a Proc object in a variable
==================================================

It's also possible to pass an existing `Proc` object to a method waiting for a block. The only thing you have to do is to use your variable prefixed by a & rather than using an inline block:

{{ highlight irb }}
proc = Proc.new { |n| puts n }
3.times(&proc)

# => 0
# => 1
# => 2
{{ endhighlight }}

That's fun but is there real use case?
======================================

For sure! Let's say you're writing a Rails app and that you need to be able to sort your blog posts by title, author, descending id or by state (published / draft). Using `Proc` could really save your some time and code lines.

Even if I know that sorting should be done at DB level for performance concerns, one of the possible implementation showing `Proc` usage is:

{{ highlight ruby }}
def sort
	order = params[:order] || "id"
	proc = case order
	when "author" then lambda { |a| a.user.name.downcase }
	when "status", "title" then lambda { |a| a.send(order) }
	when "id" then lambda { |a| -a.id }
	end

	@articles = Article.all.sort_by &amp;proc
end
{{ endhighlight }}

This action code is really easy to understand. First we're defining an *order* variable in which we'll store the sorting parameter that we've retrieved from POST or GET parameters. If there's no *order* parameter we'll use *id* by default.

Then we populate the *proc* variable with a `lambda` acording to the requested order — *lambda* is in fact a specialized *Proc* object that checks if it has the good number of parameters. If the user wants to sort by author, we'll use downcased *name* attribute. If the sorting is based on the status or the title, we'll use the attribute without any further processing. We're using an `Object#send` call to avoid duplication and conditions. The corresponding method is called dynamicly. If user wants to sort based on descending id we just have to call the corresponding method the only thing to think of is to use a minus to sort in reverse order.

When we have the good `Proc` we use ActiveRecord capabilities to fetch all records and then we sort them using `Enumerable#sort_by` method wich take a block as its parameter to know how to sort the collection.

This kind of code is not spectacular at all but I really think it can be useful in many cases. It leads to a shorter and more maintainable code. When you have a lot of duplication with small differences you really should think of `Proc` to keep your code DRY.

For example I heavily used `Proc` for one of my console software Renamer. It can be a good study case for a better understanding of `Proc`