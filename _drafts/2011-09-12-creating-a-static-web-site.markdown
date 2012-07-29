---
layout: post
title: Effectivly manage a static website
category: ruby
tags: ['ruby', 'web', 'tools', 'shell', 'static']
---

Couple months ago I was working on my [personal website](http:/www.cavigneaux.net) that I decided to re-write from scratch since most of the existing things will be scratched off and replaced.

This site has a really simple purpose: present myself, my knowledges and projects I work on. A way for potential customers & partners to know a bit more about me and my skills.

Rather than writing a full-fledge Rails app I decided to create a simple static website since most of the content will rarely change. Things that need to be fetch dynamically will use Javascript.

# Nanoc

No I was not ready to write all site pages with raw HTML / CSS again… [Nanoc](http://nanoc.stoneship.org/) was the right solution for my needs.

Nanoc is a web site generator which can use layouts and compile documents written in [HAML](http://haml-lang.com/) or [Markdown](http://daringfireball.net/projects/markdown/) into multiple directories and static files ready to be push on the web server.

Nanoc is available as a gem and can be installed this way:

{% highlight console %}
gem install nanoc
{% endhighlight %}

You're now ready to create your website skeleton:

{% highlight console %}
nanoc create_site cavigneaux.net
{% endhighlight %}


This leaves you with a new directory (cavigneaux.net in previous example) containing:

- a YAML config file
- a `Rakefile` with predefined tasks for Nanoc
- a `Rule` file which declares routes and compilation rules
- a `content` directory with an HTML index file and a default stylesheet
- a default layout in `layouts` directory

First thing to do is to try to compile this skeleton to understand what is happening under the hood:

{% highlight console %}
nanoc compile
{% endhighlight %}

It should create a new directory named `output` containing the static version of the website. You can browse thow it using the bundle web server:

{% highlight console %}
nanoc view
{% endhighlight %}

You can now browse [http://localhost:3000](http://localhost:3000). This default page was generated using files in `content` directory. It's an introduction to Nanoc.

You can now hack your site adding HTML, Javascript, CSS and images files. I decided to use HAML for layout and content files, SCSS for stylesheets and CoffeeScript for javascripts.


Here is my tweaked nanoc config:

{% highlight yaml %}
# Needed by nanoc to build absolute links
base_url: http://www.cavigneaux.net

# I want to deploy using the built-in rsync Rake task
deploy:
  default:
    dst: "user@host:/path/to/www/cavigneaux.net"
    options: ["-avz", "--delete", "-e ssh"]

# A list of file extensions that nanoc will consider to be textual rather than
# binary. If an item with an extension not in this list is found,  the file
# will be considered as binary.
text_extensions: [ 'css', 'erb', 'haml', 'htm', 'html', 'js', 'less', 'markdown', 'md', 'php', 'rb', 'sass', 'scss', 'txt', 'xhtml', 'xml' ]

# I want generated output to be in public/ rather than in output/
# It allows me to use pow as a webserver to check my local devs
output_dir: public

# A list of index filenames
# This list is used by nanoc to generate pretty URLs.
index_filenames: [ 'index.html' ]

# Don't need a diff on generation
enable_output_diff: false

data_sources:
  -
    type: filesystem_unified

    # The path where items should be mounted (comparable to mount points in
    # Unix-like systems). This is “/” by default, meaning that items will have
    # “/” prefixed to their identifiers. If the items root were “/en/”
    # instead, an item at content/about.html would have an identifier of
    # “/en/about/” instead of just “/about/”.
    items_root: /

    # The path where layouts should be mounted. The layouts root behaves the
    # same as the items root, but applies to layouts rather than items.
    layouts_root: /
{% endhighlight %}

# HAML / SCSS / CoffeeScript

# Guard

# Livereload

rb-fsevent

# Rake for deployment

rsync