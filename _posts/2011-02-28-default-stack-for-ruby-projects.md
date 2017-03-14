---
title: Default stack for Ruby projects
category: Ruby
tags: ['best practices']
header:
  overlay_image: /assets/images/headers/ruby.png
  overlay_color: "#B06713"
---

Having a clear and well-defined default stack when you start a new project is a good thing — it will have an impact on your productivity and happiness. I'll tell you which is mine and why. Your default stack may (should) vary regarding your project type. It's my case so I'll start by showing you common tools I use then we'll see what are the other tools dedicated to specific kind of projects.

The first thing I do when I start a new project is to create a project directory. I `cd` in then I setup my common tools.

Common tools
============

RVM
---

I use [RVM](http://rvm.beginrescueend.com/ "RVM: Ruby Version Manager") on every single project I work on, it allows me to choose [Ruby](http://ruby-lang.org/ "Ruby Programming Language") version I want to use on this project and I also have a separated [gemset](http://rvm.beginrescueend.com/gemsets/ "RVM Gemsets"). From within my project directory I only see gems related to this project (and global ones).

I often need to switch Ruby version from a project to another one so RVM is really the perfect tool in my workflow. Old projects use Ruby 1.8, all new projects are using Ruby 1.9 and most of Rails projects are using [REE](http://www.rubyenterpriseedition.com/ "Ruby Enterprise Edition").

Ruby 1.9
--------

Even if I use RVM, most of the time I prefer to use ruby 1.9.x ([MRI](http://en.wikipedia.org/wiki/Ruby_MRI "Matz's Ruby Interpreter")) because:

- it's the reference implementation
- it has pretty good performances
- it includes many cool things ruby 1.8.x doesn't have

Bundler
-------

[Bundler](http://gembundler.com/ "Application Dependencies Manager") is a must-have in any project. Your project dependencies will be clearly defined and you'll ease kickstart hacking for contributors.

MiniTest
--------

[MiniTest](http://bfts.rubyforge.org/minitest/) provides a complete suite of testing facilities supporting TDD, BDD, mocking, and benchmarking. MiniTest is now the default testing framework shipped with Ruby 1.9.

MiniTest has a real value because without any other third-party lib you can do:

- [TDD](http://en.wikipedia.org/wiki/Test-driven_development) with minitest/unit which is a small and fast.
- [BDD](http://en.wikipedia.org/wiki/Behavior_Driven_Development) with minitest/spec which hooks onto minitest/unit to bridge test assertions over to spec expectations.
- Mocks with minitest/mock a tiny mock object framework
- Benchmarks with minitest/benchmark to assert the performance of your code
- Autorun of tests when a file change
- Test randomization to ensure your tests don't accidentally becoming order-dependent
- Colored output

MiniTest can even be use with Ruby 1.8 and 1.6.

SimpleCov
---------

[SimpleCov](https://github.com/colszowka/simplecov) is a code coverage analysis tool for Ruby 1.9. It uses 1.9’s built-in Coverage library to gather code coverage data, but makes processing it’s results much easier by providing a clean API to filter, group, merge, format and display those results, thus giving you a complete code coverage suite that can be set up with just a couple lines of code.

Enabling it in your code is easy as :

{% highlight ruby %}
require 'simplecov'
SimpleCov.start
{% endhighlight %}

SimpleCov can be use with any test framework and will output nice formatted HTML pages.

YARD
----

[YARD](http://yardoc.org/ "YARD - A Ruby Documentation Tool") is a documentation generation tool with:

- a local documentation server with dynamic searching and live reloading
- easy customization
- extensible
- meta-tags to describe parameters, types, return value type, exception, deprecation, …
- RDoc compatibility


[RubyDoc](http://rubydoc.info/ "RubyDoc.info - Library Listing") hosts documentation for all gems available on [RubyGems](http://rubygems.org/ "RubyGems.org | Community gem host") and requested [GitHub](https://github.com/) projects.

Rake / Thor
-----------

Most of the things you do when you're not effectively coding can be automated. That's why I like to have a Rakefile for project specific tasks or / and a Thorfile for system-wide tasks.

[Rake](http://rake.rubyforge.org/ "Rake -- Ruby Make") is a simple ruby build program with capabilities similar to make. It can create tasks, dependencies, …

[Thor](https://github.com/wycats/thor) is an efficient tool for building self-documenting command line utilities. It simplifies parsing of command line options, banners writing, and can be used as an alternative to Rake. The syntax is Rake-like, so it should be familiar to most Rake users. Thor can define local and system-wide tasks.

Mercurial
---------

[Mercurial](http://mercurial.selenic.com/ "Mercurial SCM") (hg) is my DSCM of choice. It integrates seamlessly in my workflow and all the tasks I have to do with it are naturals. IMHO Mercurial is much more easier to use than Git for simple versioning usage. Mercurial is full-features DSCM:

- cross-platform
- easy to learn and use
- fast
- can handle project of any size
- multiple workflow styles available
- extensible
- integrated in TextMate

When I work on an open-source project I want its code (or content) to be freely and easily available for reading or contributing. That's were [Bitbucket](https://bitbucket.org/) comes in. In short it's a [GitHub](https://github.com/) for Mercurial users. But in fact, when you look closer you can see that it's not the same community as Githubers and that conventions and defaults are not the same on the two sides.

I'm really a big fan of Bitbucket. All my projects are hosted on it and benefit of an issue tracker, a wiki, permissions management, sources and an API support. You can follow a project, fork it, send pull requests and so on…

To be honest I'm a big fan of Mercurial and can't work effectively with anything else. [Git](http://git-scm.com/ "Git - Fast Version Control System") is great but way to complicated and non-intuitive for my slow brain.

Command-line
============

For really simple repetitive tasks (copy / remove files, generate documentation, start tests, package a gem) I tend to use Rake or Thor. It's straight-forward and easy to use. For most advanced command-line tools I use my default stack and add gems when I need it.

Small websites and API
======================

When I need a simple website (no much interaction with the user, simple features, showcases, …). I prefer to use a lightweight framework. API should be backed by fast and light framework too. That's why most of the time I use [Sinatra](http://www.sinatrarb.com/ "Sinatra") over another alternative like [Rails](http://rubyonrails.org/ "Ruby on Rails").

Sinatra
-------

Sinatra is a DSL for quickly creating web applications in Ruby with minimal effort. You can use it to prototype your ideas, quickly put an API online, … Sinatra is very simple to learn since it doesn't involve a lot of magic. It keeps things simple but evolutive.

Here is a working full example of an "Hello World" app:

{% highlight ruby %}
require 'rubygems'
require 'sinatra'

get '/' do
 "Hello World!"
end
{% endhighlight %}

5 lines so I think you understand now why Sinatra is especially well-fitted for creating APIs. Sinatra is [Rack](http://rack.rubyforge.org/ "Rack: a Ruby Webserver Interface") bases app so it's easily extensible.

Rack::Bug
---------

Rack::Bug adds a diagnostics toolbar to Rack apps. When enabled, it injects a floating div allowing exploration of logging, database queries, template rendering times, etc. I'm nearly always watching my term to read logs, exceptions, SQL query and times, …

[Rack::Bug](https://github.com/brynary/rack-bug) gives you many information right in your browser:

- Rails Info
- Timer
- Request Variables
- SQL
- Active Record
- Cache
- Templates
- Log
- Memory

Here's a screenshot:

[![Rack::Bug Screenshot](http://img.skitch.com/20091027-n8pxwwx8r61tc318a15q1n6m14.png "Rack::Bug screenshot")](http://img.skitch.com/20091027-n8pxwwx8r61tc318a15q1n6m14.png "Big screenshot")

HAML
----

[HAML](http://haml-lang.com/ "#haml") is a clean, fast and efficient way to write HTML.

SASS
----

[SASS](http://sass-lang.com/ "Sass - Syntactically Awesome Stylesheets") is an extension of CSS3, adding nested rules, variables, mixins, selector inheritance, and more. It’s translated to well-formatted, standard CSS using the command line tool or a web-framework plugin.

Webapps
=======

Rails
-----

For real webapps with lots of user interactions, features and a database I always use [Rails](http://rubyonrails.org/ "Ruby on Rails"). As often as I can I use Rails 3 since it brings many great enhancements. You shouldn't start a new webapp project with Rails 2.x.

**Gems**

- Rack::Bug — [see above](#rackbug)
- [FlashHelper](https://rubygems.org/gems/flash_helper) — a gem which ease flash notifications displaying
- [MetaWhere](http://metautonomo.us/projects/metawhere/ "MetaWhere  &#8211;  metautonomo.us") — ActiveRecord finder on steroïds with pure Ruby SQL syntax
- [Devise](https://github.com/plataformatec/devise) — Flexible authentication based on Warden
- [Cancan](https://github.com/ryanb/cancan) — Authorization which restricts what resources a given user is allowed to access
- [Kaminari](https://github.com/amatsuda/kaminari) — a scope & engine based paginator
- HAML — [see above](#haml)
- SASS — [see above](#sass)
- [CarrierWave](https://github.com/jnicklas/carrierwave) — a simple and flexible way to upload files

Here is a great blog post about [how to setup Devise and Cancan](http://www.tonyamoyal.com/2010/07/28/rails-authentication-with-devise-and-cancan-customizing-devise-controllers/ "Rails Authentication with Devise and CanCan &#8211; Customizing Devise Controllers | Tony Amoyal").

MooTools
--------

To improve user experience, make my webapp feels more friendly and responsive I use javascript. Most of the time I use an existing javascript framework to ease my work. I know that it's trendy to use [jQuery](http://jquery.com/ "jQuery: The Write Less, Do More, JavaScript Library") but I really prefer [MooTools](http://mootools.net/ "MooTools - a compact javascript framework"). Maybe you're thinking "What a shame loser!" but it's the only framework that integrates seamlessly in my workflow, it does every single thing I need, there's a great community, great libs and I'm fond of MooTools' way of thinking and syntax.

MooTools is a compact, modular, Object-Oriented JavaScript framework. It allows you to write powerful, flexible, and cross-browser code with its elegant, well documented, and coherent API.

MooTools code respects strict standards and doesn't throw any warnings. It's extensively documented.

If you don't know MooTools and use jQuery or [Prototype JS](http://www.prototypejs.org/ "Prototype JavaScript framework: Easy Ajax and DOM manipulation for dynamic web applications") only because it's the default framework you should take a look at MooTools. It's really worth it.

GUI
===

At home I only have Mac computers. I can't stay more than 30 minutes in front of a Windows © screen. I love UI and system integration of Cocoa apps. Most Mac apps are using Cocoa and Objective C as their programming language / framework.

Objective C is a great and interesting language to use but I'm still in love with Ruby. Here comes [MacRuby](http://www.macruby.org/ "MacRuby &raquo; Home").

MacRuby
-------

MacRuby is an implementation of Ruby 1.9 directly on top of Mac OS X core technologies such as the Objective-C runtime and garbage collector, the LLVM compiler infrastructure and the Foundation and ICU frameworks. It is the goal of MacRuby to enable the creation of full-fledged Mac OS X applications which do not sacrifice performance in order to enjoy the benefits of using Ruby.

You can now create Mac OS X native apps using Cocoa from within Ruby without any third-party bridge. Moreover Ruby and MacRuby projects are well integrated in Xcode. You benifit from Objective C garbage collection which is fast,  and no GIL so you can do real threaded algorithms. [GDC](http://en.wikipedia.org/wiki/Grand_Central_Dispatch "Grand Central Dispatch - Wikipedia, the free encyclopedia") can be used for easy task parallelism.

Last thing about MacRuby to notice — it's an official [Apple](http://www.apple.com/ "Apple") sponsored project. [Lead developer](http://chopine.be/ "Laurent Sansonetti") is a Ruby guru and works at Apple.

What about you?
===============

I'd really like to know what you're using and why you chose a different stacks so please feel free to leave a comment on this post.
