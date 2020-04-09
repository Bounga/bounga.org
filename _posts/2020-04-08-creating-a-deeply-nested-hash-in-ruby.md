---
title: Creating a deeply nested Hash in Ruby
category: Ruby
tags: ['tips', 'ruby']
header:
  overlay_image: /assets/images/headers/ruby.png
  overlay_color: "#4D4D4D"
excerpt: "Sometimes you'll have to create a deeply nested hash without
    knowing how deep it can be at first. This hash should allow
    reading and setting values at any level without even having to
    manually create its ancestors if missing. Let's see how to do this."
---

Ruby is a very dynamic programming language. Here is a quick
demonstration of this fact.

It's not something I need on a daily basis but it happens that I want
to be able to create a hash in which I could set or read the value for
an arbitrary deep key without even knowing if its ancestor keys
exists.

# Common hash usage #

## Problem ##

Let's say I have a variable named `hash` that contains a hash:

```irb
irb> hash = {}
```

and I want to know what is the value for `hash[:foo][:bar][:baz]`:

```irb
irb> hash[:foo][:bar][:baz]
NoMethodError: undefined method `[]' for nil:NilClass
```

That was pretty predictable. But how do I read these nested values?

To go further, how do I assign a value to some deep key?

```irb 
irb> hash[:foo][:bar][:baz] = "hello"
NoMethodError: undefined method `[]=' for nil:NilClass
```

The same kind of exception is raised.

## Solution ##

It's time to create this deep hash by hand and it's gonna be painful:

```irb
irb> hash = {}
=> {}
irb> hash[:foo] ||= {}
=> {}
irb> hash[:foo][:bar] ||= {}
=> {}
irb> hash[:foo][:bar][:baz] = "hello"
=> "hello"
irb> hash
=> {:foo=>{:bar=>{:baz=>"hello"}}}
```

I'm pretty sure you don't like to write this kind of thing in your
code, do you?

When I'm facing something like that I can't stop thinking that there's
a better way to do it since Ruby was designed with programmers
happiness in mind.

# Next level hash usage #

Obviously there's a better way!

Most of the time in Ruby we create our hashes by using the syntax we
saw above. That's fine for common needs but you must know that this
syntax is only a syntactic sugar to quickly create a hash but under
the hood it's a call to the `Hash.new` method.

If you read `Hash.new` documentation carefully you'll notice that
there is a signature that allows to pass a block to the `Hash.new`
call.

This block let you choose how the hash should act when you try to
access a key that doesn't exist. This is exactly what we need to
implement our infinitely deep nested hash!

## Making a hash dynamic ##

Let's do it:

```irb
irb> hash = Hash.new { |h, k| h[k] = h.dup.clear }
=> {}
irb> hash[:foo][:bar][:baz] = "test"
=> "test"
irb> hash["access"]["only"]
=> {}
irb> hash
=> {:foo=>{:bar=>{:baz=>"test"}}, "access"=>{"only"=>{}}}
```

Much better!

As you can see we can now assign a value to an arbitrarily deeply
nested key in the hash without having to take care of creating
ancestors. That's pretty awesome, isn't it?

You may ask what's the sorcery done in our example `new` block?

This is kind of recursion in a way. By using `h[k] = h.dup.clear` we
are telling that if the key `k` is missing we want to duplicate `h`,
which is our main hash, then clear its content and store it as the
value of the `k` key.

It can seem a bit weird but actually it's not. This cloning / clearing
black magic creates a new dynamic hash as the value for this key by
reusing our main hash behavior.

## Gotcha ##

As you have may notice there's a gotcha you have to be aware of when
using this trick.

As soon as you'll try to access a given key in the hash, it will
create the key with content being what you've defined in the `new`
block.

That's what happened with the `hash["access"]["only"]` call in our
example.

So if you assign a value to a non-existing key, this key and all its
missing ancestors will be created and the value will be associated to
the key.

But if you try to read the value of a non-existing key, without
setting its content, then this key will be created with a value
defaulting to what was defined in the block. In our example it's going
to be an empty hash.

Hope that this little tip will help you someday. Have fun and stay
safe.
