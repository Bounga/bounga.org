---
layout: post
title: Easily add a lightweight logging system to your Ruby app
category: ruby
tags: ['ruby', 'debug', 'tips', 'logging']
---

Sometimes it can be very useful to add verbose and silent modes to your app. This way users can easily control information level about events happening. As you can figure out, these modes are especially useful for command-line apps.

I'm aware that there are robust solutions available out there such as Log4R but I don't think you always need such a complete tool for a simple app with simple needs. The pros of my solution are to be very lighweight and doesn't require any third-party lib. If you're writing your app using [Ruby](http://www.ruby-lang.org) you can use core properties to do this.
    
When a Ruby interpreter is started, it automatically create some global variables. The one which is catching our attention is `$VERBOSE`. By default, this variable is set to `false`.

`Kernel#warn()` method from the [standard library](http://apidock.com/ruby/Kernel/warn) allows us to print a message on the standard error output (STDERR). This method only prints the given string if `$VERBOSE` variable is not `nil`. Having our verbose mode requires to use `Kernel#warn()` method rather than `Kernel#puts()` when we want to print a warning, an error message, etc.

Let's say that by default, our app is running in silent mode. Our code must set `$VERBOSE = nil` before any `Kernel#warn()` method call. To give the user more details, we can for example use the command-line arguments to set it.

[*OptParse*](http://apidock.com/ruby/OptionParser) is a good choice for handling command-line arguments. More over it's included by default in the Ruby interpreter.

The needed code is really simple:

{% highlight ruby %}
OptionParser.new do |opts|
  opts.on("-V", "--verbose", "Activate verbose mode") { $VERBOSE = true }
  opts.on("-h", "--help", "Display this help" ) { puts opts; exit; }
end
{% endhighlight %}

We only changed one variable value to activate the requested level of verbosity.

So with just one line of code you'll be able to have a really ligthweight, simple and useful logging system.