---
title: Pow using rbenv
category: ruby
tags: ['ruby', 'webserver', 'pow', 'rbenv']
---

Couple weeks ago I switched from [RVM](http://beginrescueend.com/) to [rbenv](https://github.com/sstephenson/rbenv) because I think it's a cleaner and faster way to handle Ruby versions. It is more respecful of the Unix philosophy.

It worked like a charm right after the installation but [Pow](http://pow.cx/) wasn't working anymore. There's just a little trick to know.

So let's have a reminder about rbenv and see how to make it work with Pow.

Daily use
=========

I'm running Mac OS X and chose to use [HomeBrew](http://mxcl.github.com/homebrew/) as my package manager. Let's install rbenv and Pow.

Rbenv Install
-------------

{% highlight console %}
$ brew update
$ brew install rbenv
$ brew install ruby-build
$ brew install pow
{% endhighlight %}

You're now ready to install your rubies. Be sure to **uninstall RVM first** if you installed it. RVM overwrites many shell commands and would cause troubles to rbenv if there are both installed:

{% highlight console %}
$ rvm implode
{% endhighlight %}

Add this line to your *.bash_profile* or *.zshenv* :

{% highlight bash %}
export RBENV_ROOT=/usr/local/var/rbenv
if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi
{% endhighlight %}

Open up a new shell and you're ready to go.

Rubies install
--------------

First you can list candidates for installation:

{% highlight console %}
$ rbenv install
{% endhighlight %}

Install the one you need:

{% highlight console %}
$ rbenv install 1.9.2-p290
{% endhighlight %}

Check installed versions:

{% highlight console %}
$ rbenv versions
{% endhighlight %}

Specify a default Ruby version to use system-wide:

{% highlight console %}
$ rbenv global 1.9.2-p290
{% endhighlight %}

Specify an other version to use in a given directory / project:

{% highlight console %}
$ cd Code/foo
$ rbenv local ree-1.8.7-2011.03
{% endhighlight %}

You can also set a ruby version to use in a shell for its lifetime:

{% highlight console %}
$ rbenv shell 1.9.3-p0
{% endhighlight %}

Gemsets
=======

rbenv users tends to think that [Bundler](http://gembundler.com/) is the way to go to handle project dependencies. Why use a gemset when the project itself can be the gemset. If you have a *Gemfile* then you can do:

{% highlight console %}
$ bundle install --path vendor/bundle
{% endhighlight %}

Gems will be installed in *vendor/bundle* project directory. Be sure to exclude this dir from your SCM.

You can automate this behaviour by using Bundler config file. Edit *~/.bundle/config* like so:

{% highlight yaml %}
---
  BUNDLE_PATH: vendor/bundle
{% endhighlight %}

If you really need and want something similar to RVM gemsets, take a look at [rbenv-gemset](https://github.com/jamis/rbenv-gemset).

Using Pow with rbenv
====================

***Update:** Note for users of Pow < 0.4.2 only. As of Pow >= 0.4.2 the following problem doesn't exist anymore since Pow now loads user's environment variables properly for Zsh users.*

I ran into a problem when I tried to access my projects using Pow. Pow was getting the request but crashes due to gems missing or bad ruby version.

Pow doesn't know anything about rbenv so you need to tell him about the right PATH to be able to use rbenv.

Edit *~/.powconfig* like this:

{% highlight bash %}
export PATH=$(rbenv root)/shims:$(rbenv root)/bin:$PATH
{% endhighlight %}

Be sure to restart pow after editing this file.

Now you have rbenv and Pow up & running!
