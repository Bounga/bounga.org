---
layout: post
title: Googlus - A Ruby wrapper around Google URL Shortener API
category: lib
tags: ['ruby', 'google', 'url', 'shortener', 'api', 'wrapper', 'lib', 'software', 'oss', 'release']
---

[Googlus](https://bitbucket.org/Bounga/googlus/) is a [Ruby](http://www.ruby-lang.org) wrapper around [Google URL Shortener REST API](http://code.google.com/intl/fr/apis/urlshortener/overview.html).

I often use URL shortener services and I was like "It would be cool if I could use it in Textmate, in my Ruby code or on the command-line!" so I wrote this thin lib to be able to do that.

Features
========

- shorten URLs
- expand URLs
- analyse URLs usage

Installation
============

{% highlight bash %}
$ gem install googlus
{% endhighlight %}

Usage
=====

In your Ruby code:

{% highlight ruby %}
require 'rubygems'
require 'googlus'

Googlus::Shorten.new(long_url).short_url
Googlus::Expand.new(short_url).url
Googlus::Analytic.new(url).to_s
{% endhighlight %}

On the command-line:

{% highlight bash %}
$ googlus -s http://www.bounga.org # Shorten URL
$ googlus -e http://goo.gl/arOIJ   # Expand short URL
$ googlus -a http://goo.gl/arOIJ   # Get analytics for short URL
{% endhighlight %}
  
Other
=====

If you want to contribute you should take a look at:

- [Source Style](http://www.bitbucket.org/Bounga/googlus/wiki/SourceStyle)
- [Ticket Guidelines](http://www.bitbucket.org/Bounga/googlus/wiki/TicketGuidelines)
- [Contributing](http://www.bitbucket.org/Bounga/googlus/wiki/Contributing)

[Source](http://www.bitbucket.org/Bounga/googlus/src) can be cloned using [Mercurial](http://mercurial.selenic.com/) on my [Bitbucket repo](https://bitbucket.org/Bounga/googlus).

Problems, comments, and suggestions are welcome on the [tracker](http://www.bitbucket.org/Bounga/googlus/issues/new/).

This gem is released under the [MIT license](http://creativecommons.org/licenses/MIT/).