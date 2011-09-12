---
layout: post
title: Thor - a command-line scripting tool
category: ruby
tags: ['ruby', 'gem', 'command-line', 'admin', 'shell', 'thor', 'rake']
---

[Thor](https://github.com/wycats/thor) is a simple and efficient tool for building self-documenting command line utilities. I'm a shell guy, I use a terminal everyday and most of my job is done in my [favorite text editor](http://macromates.com) or in a terminal. To be effective I need to have a suitable text editor and the ability to automate most of the things I do in the terminal. 

Here comes Thor. Thor will help you to script your common actions and command-lines. Rather than having five commands to do a recurent job why just not having one?!

Thor will remove the pain of having to handle command-line options parsing, documentation and tasks dependencies. You'll be able to use Thor on a given project or even system-wide. It's a system-wide [Rake](http://rake.rubyforge.org/) on steroïds which ease command-line options parsing, inline help generation, file-system manipulation, templating, …

This post will show you how you can use Thor to simplify your daily tasks. We'll see all Thor capabilities and show some simple but real-world use cases. 

Last but not least, Thor was written by [Yehuda Katz](http://yehudakatz.com/).

Let's go ! Install it via `gem install thor`

# First task

To create tasks you need a *Thorfile* (Rakefile equivalent) which will include our tasks code.

A simple example would be:

{% highlight ruby %}
class Test < Thor
  desc "hi", "Say hi"
  def hi
    puts "hi !"
  end
end
{% endhighlight %}

Now you can try `thor list` ou `thor -T` to see available tasks:

    test
    ----
    thor test:hi  # Say hi

# Add parameters

Tasks can take arguments. This way you don't need to use environment variables anymore as you're used to with Rake:

{% highlight ruby %}
class Test < Thor
  desc "hi NAME", "Say hi"
  def hi(name)
    puts "Hi #{name} !"
  end
end
{% endhighlight %}

Now help displays:

    $ thor help test:hi

    Usage:
      thor test:hi NAME

    Say hi

Let's try our new task:

    $ thor test:hi Nico

    Hi Nico !

What we've defined is a *parameter* so it is mandatory. If you forget a mandatory parameter, Thor will complain about it:

    "hi" was called incorrectly. Call as "thor test:hi NAME".

# Adding options

You'll often need "optionnal parameters". Options can be added this way:

{% highlight ruby %}
class Test < Thor
  desc "hi NOM", "Say hi"
  method_option :verbose, :aliases => "-v", :desc => "Be verbose"
  def hi(name)
    puts "Hi #{name} !"
    puts "How are you today?" if options[:verbose]
  end
end
{% endhighlight %}

Give it a try:

  $ thor test:hi Nico -v

  Hi Nico !
  How are you today?

Options can be defined with a default, given type, …:

    method_option :path, :type => :string, :default => "~/", :required => false, :aliases => "-p"

Available types are:

{% highlight ruby %}
:boolean # --option / --option=true / --no-option
:string  # --option=VALUE
:numeric # --option=2
:array   # --option=nico matz dhh
:hash    # --option=nom:Nico lang:ruby
{% endhighlight %}

# Add task dependencies

Thor allows to invoke a task from within another task so you can chain task calls and define dependencies:

{% highlight ruby %}
class Counter < Thor
  desc "one", "Prints 1"
  def one
    puts 1
  end

  desc "two", "Prints 1, 2"
  def two
    invoke :one
    puts 2
  end

  desc "three", "Prints 1, 2, 3"
  def three
    invoke :two
    puts 3
  end
end
{% endhighlight %}

`Thor::Group` class can achieve the same goal calling each method of the group, one by one, in definition order:

{% highlight ruby %}
class Counter < Thor::Group
  desc "Prints 1 2 3"

  def one
    puts 1
  end

  def two
    puts 2
  end

  def three
    puts 3
  end
end

# $ thor counter
# => 1
# => 2
# => 3
{% endhighlight %}

# Interacting with user

Thor bundles some methods which ease interacting with user. Here are some:

- say
- ask
- yes?
- no?
- add_file
- remove_file
- copy_file
- template
- directory
- inside
- run
- inject_into_file

To be able to use it you'll need to include `Thor::Actions` in your class.

You should really take a look at [these methods documentation](http://rdoc.info/github/wycats/thor/master/Thor/Actions) which will save you some time.

# Namespaces

Sometimes you'll need to have your tasks defined in a namespace to ensure there's no collision with other tasks. No problem:

{% highlight ruby %}
module Bounga
  class App < Thor
    # tasks
  end
end

# $ thor bounga:app:task
{% endhighlight %}

# System-wide !

One the most interesting feature of Thor is that a task can be installed system-wide. It means that you don't need to have a Thorfile in your current directory to be able to call a task :

    thor install Thorfile

You can now use tasks defined in your Thorfile from anywhere!

# Real-world example

Let's say your working on a Rails app you often need to clone and setup maybe because there's a lot of developers on the project or maybe you need multiple repos to test things … 

{% highlight ruby %}
class Setup < Thor
  desc "prepare", "copy configuration files"
  method_options :force => :boolean
  
  def prepare(file = "*.dist")
    Dir["config/#{name}"].each do |source|
      destination = "config/#{File.basename(source, ".dist")}"
      FileUtils.rm(destination) if options[:force] && File.exist?(destination)
      
      if File.exist?(destination)
        puts "Skipping #{destination} because it already exists"
      else
        puts "Generating #{destination}"
        FileUtils.cp(source, destination)
      end
    end
  end

  desc "populate", "generate records"
  method_options :count => 10
  
  def populate
    require File.expand_path('config/environment.rb')
    options[:count].times do |num|
      puts "Generating post #{num}"
      Post.create!(:title => "Post #{num}", :body => "Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.")
    end
  end
end
{% endhighlight %}

You can easily extend this example and use it everywhere by installing it system-wide rather than adding the same Rakefile to each project.

Such a framework allows you to easily create automated tasks, generators and more. It can even be a basis for a command-line interface to one of your lib and saves you from writing boring command-line parser.

You should really give it a try if you find yourself doing the same thing again and again in your terminal. It's super easy to setup and all needed tools are bundled. If you want to improve and speed up your workflow, Thor will help you for sure.