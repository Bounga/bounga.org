---
layout: post
title: Understand the lookup path in Ruby
category: Ruby
tags: ['tips', 'meta-programming']
excerpt: "Do you know how method lookup works in Ruby? Did you ever
wonder how the interpreter can handle mixin, multiple definitions of
the method? That's what we'll cover today."
---

In Ruby, whenever an object gets a message, runtime searches for a
method holding this name in the object class then its super classes or
in a module that was mixed in one of these classes.
      
But do you know how this works? How ambiguous cases are handled, for
example a method name existing in both the object class and a mixin?
Which one will be run?

As always, understanding details of how Ruby works will help you to
write cleaner, concise but also more powerful code.

So let's get our hands dirty and see how the lookup path is working.

Don't be afraid the lookup process is pretty easy to understand. To
demonstrate how Ruby choose which method should be executed we're
gonna write simple code with methods that only print a message so
we'll be able to trace everything.

Go ahead and write some code that will use everything we can:

{% highlight ruby %}
module M
    def print_it
        puts "print this from M module"
    end
end

class C
    include M
end

class D < C
end

obj = D.new
obj.print_it
{% endhighlight %}

At the end we're instantiating an object based on the `D` class then
we call its `print_it` method. When the Ruby interpreter analyze this
it can tell there's no such method in the `D` class so it checks if
this class mixes some module but it don't. The interpreter then checks
if one of the superclasses (`C`) implements such a method. No luck, no
method definition is matching in the superclasses.

It then checks if the superclass is mixing a module that implement
this method and this time it can find it (`M` module is mixed in `C`
class)! As it is available it uses it.

This is a very typical way for the Ruby interpreter to search for a
given message in the available methods on a given object.

## How deep can the lookup goes? ##

As you may know, in Ruby, every object is an instance or a descendant
of the `Object` class. So the interpreter will always try to go up
until this class if no method matches on the previous class. 

It can even go further since the `Object` class mixes `Kernel` module.
`Kernel` module is the last door keeper. It is responsible for
handling things like `respond_to?` or `send` which are available in
all our objects.

If the interpreter goes up until the `Kernel` module and that the
method still hasn't been found the the interpreter will give up and
throw a `NoMethodError` exception.

## What about mixing in many modules defining the same methods ? ##

It's still really simple, Ruby applies the principle of least
surprise. Let's say we're writing this:

```ruby
module M
    def print_it
        puts "In M"
    end
end

module N
    def print_it
        puts "In N"
    end
end

Class C
    include M
    include N
end

obj = C.new
obj.print_it
# => "In N"
```

Ruby does what you would expect, the last definition of the given
method takes precedence. That's a thing I like a lot in Ruby, it's
predictable.

## And what if you want to force to look up in the path? ##

You know what? You can do that? You just have to use the `super`
keyword to explicitly tell you want to use the parent definition.

By doing so you're telling the interpreter that you know what you're
doing and you want to search for a method definition that is located
in the parent or its ancestor even if there's a matching method in the
current scope.

This is especially useful when you have an ancestor method that does
pretty much all the job but lacks of some details. In this case you
can write the specific stuff then call the ancestor method as in:

```ruby
class Bicycle
    attr_reader :wheels, :seats
    def initialize
        @wheels = 2
        @seats = 1
    end
end

class Tricycle < Bycicle
    def initialize
        super
        @wheels = 3
    end
end
```

`super` is useful when you want to use the default behavior of an
ancestor then change some defaults. In our example we did initialize
everything for our tricycle based on a bicycle but then we changed the
number of wheels.

This is a good way to avoid duplication.

## To conclude ##

In my opinion it is really important to understand how the interpreter
works to search for a given method name. If you can understand how it
works you'll be able to build more powerful ode with much less
redundancy.

When you'll be comfortable with the way the interpreter search for a
given method you'll be able to use inheritance and modules at there
best.
