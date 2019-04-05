---
title: Rails console tips
category: Tips
tags: ['rails']
header:
  overlay_image: /assets/images/headers/tips.jpg
excerpt: "Knowing how to use the Rails console can be very useful when
debugging or trying your snippets. Being proficient with it will
increase your productivity."
---

When you're using [Ruby or Rails](https://rubyonrails.org) you'll have
to use <kbd>irb</kbd> or <kbd>rails console</kbd> which is an
<kbd>irb</kbd> session with Rails environment loaded in it.

This is an interactive console in which you can evaluate
[Ruby](https://www.ruby-lang.org/) expressions and access to your
project code.

Some of the tricks below will work only in Rails console but some will
also work in a regular Ruby console.

## Pretty printing objects ##

When you're trying some code or debugging your app you'll often need
to look at the content of some variables. The default logging in IRB
is not so great and readable:

```ruby
>> u = User.first
#<User id: 1, email: "nico@bounga.org", encrypted_password: "$2a$10$qcytDGIYGF727KorhZJJ.Oe3gCjgVosK53IqAqJkXnmz...">
```

But you can have a much more readable format by using an helper that
will display content by serializing it to [YAML](http://yaml.org):

```ruby
>> y u
--- !ruby/object:User
attributes:
  id: 1
  email: nico@bounga.org
  encrypted_password: "$2a$10$qcytDGIYGF727KorhZJJ.Oe3gCjgVosK53IqAqJkXnmzNaYOPoGlO"
=> nil
```

I really think it's a much more readable format especially for big objects.

## Last expression value ##

One major issue of my brain in IRB is to forget or at least not
anticipate that I'll need the result of an evaluation available in
variable to reuse it later. Luckily IRB offer a way to retrieve latest
evaluation result so you can do:

```ruby
>> 1 + 1
=> 2
>> res = _
=> 2
>> res
=> 2
```

## Reload latest version of code ##

When working on a Rails project you'll often find yourself using a
console to test things then modify app code then want to try it again.
When you do this you need to reload code between edits to benefit from
it in the Rails console. To do this you can quit and start again the
rails console or even better use code reloading:

```ruby
>> reload!
Reloading...
=> true
```

Now you're using the latest version of your code. Be careful, if you
already stored model instances in variables you should load them again
or you'll work on older version of the objects.

## Search for old commands ##

I often need to search for previous commands I already used that I
want to evaluate again or change then evaluate. Rather than
remembering everything and typing it again and again I use `C-r` to
search for command I already evaluated and use it as a basis.

## Auto-completion ##

Auto-completion is a must have and a feature you should use everywhere
it's available. You should use it in your editor, in your shell but
also in IRB. You can do it so take advantage of it!

Let say you're in an IRB session and you type `Acti` you can then type
`TAB` to see all available completions, choose one and complete it. It
also works with methods for example:

`"foo".to_` then `TAB` will give you something like:

```
>> "foo".to_
.to_c                   .to_dragonfly_unique_s  .to_json                .to_query               .to_sym
.to_d                   .to_enum                .to_json_raw            .to_r                   .to_time
.to_date                .to_f                   .to_json_raw_object     .to_s                   .to_yaml
.to_datetime            .to_i                   .to_param               .to_str                 .to_yaml_properties
```
    
## Helpers are availables in Rails console ##

Sometimes when you're in a Rails console you'll want to try out some
helpers you wrote earlier. You can do this through the `helper` method
that exposes all your helpers in the console:

```ruby
>> helper.number_to_currency(1.50)
=> "$1.50"
```

Combined with `reload!` it's really useful to try your helper methods.

## Requesting end-points ##

One other thing you'll often want and need to try is your end-points
whether they are JSON or old-good HTML ones. You could want to check
if the response code is the good one, if the body contains a given
string, if it redirects to something, if flash is set, …

```ruby
>> app.get "/movies"
=> 200
```

Getting `/movies` was successful.

```ruby
>> app.get "/users/1"
=> 302

>> app.response.redirect_url
=> "http://www.example.com/session/new"
```

Getting `/users/1` ended in a redirection that redirects us to
`http://www.example.com/session/new`.

```ruby
>> app.post "/session", email: 'fred@example.com', password: 'secret'
=> 302

>> app.response.redirect_url
=> "http://www.example.com/users/1"

>> app.session[:user_id]
=> 1
```

Posting to `/session` results in a redirection to the user detail page
and checking the session shows up that the session `user_id` key
contains the logged in user id.

Let's say you want to know if there's any flash available:

```ruby
>> app.flash
=> {}
```

You could also check for assigns in the action:

```ruby
>> app.assigns[:users]
=> nil
```

Or even cookies:

```ruby
>> app.cookies
=> {"_demo_session_id"=>"22ae45eb22ae45eb22ae45eb"}
```

## Sandboxing ##

Another wonderful thing in the Rails console is that you can start an
IRB session in sanboxed mode so when you quit your session everything
is rolled back. It's especially useful in production so you can try
thing being sure that everything will be in the same state as when you
started your session.

```ruby
$ rails console production --sandbox
Loading production environment in sandbox
Any modifications you make will be rolled back on exit
>>
```

As indicated, any database changes you make will be rolled back when you exit the session.

Let’s try to delete a user:

```ruby
>> User.destroy(1)

>> User.find(1)
ActiveRecord::RecordNotFound: Could not find User with id=1

>> exit
   (12.3ms)  ROLLBACK
```

If you start a new IRB session and try to find the user with id `1`
you'll find it as if nothing happened before. Really cool.

## Searching for the sources ##

Another thing you'll often need to know when you're in an IRB session
is where a given method was defined.

You can easily know this:

```ruby
>> user = User.first
>> user.rating
```

Let's say you want to know where the `rating` method is defined (even
if it pretty obvious), Ruby will help you to find out. You only have
to use:

```ruby
>> User.instance_method(:rating).source_location
=> ["/Users/nico/demo/app/models/user.rb", 39]
```

You can even go further by looking at the sources of a given method by
using `User.instance_method(:rating).source.display`.

## Configuring IRB for next sessions ##

The last thing you need to know about IRB / Rails console is that you
can configure it and be in a given state when it starts. Here is what
I do in my `~/.irbrc` config file:

```ruby
require 'irb/completion'
require 'pp'
IRB.conf[:AUTO_INDENT] = true
IRB.conf[:PROMPT_MODE] = :SIMPLE

begin
  require "pry"
  Pry.start
  exit
rescue LoadError => e
  warn "=> Unable to load pry"
end
```

In my config file I ensure completion will be available, then I
require `pp` to be able to display objects nicely in the console to
ease reading. I also make sure that IRB will indent code for me since
I'm used to read indented code. Then I change IRB prompt to a simple
one because I like to have a minimal prompt in IRB.

Then if Pry is available I load it.

## Final words ##

IRB, and by extension Rails console, is a real gem and can help you a
lot everyday. You really should try to take the best out of it. Maybe
my next blog post will be the same kind of tips but for <kbd>IEx</kbd> which is
the same kind of tool but for [Elixir](https://elixir-lang.org).
