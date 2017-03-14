---
title: Effectivly manage a static website with Nanoc
category: ruby
tags: ['ruby', 'web', 'tools', 'shell', 'static', 'nanoc']
---

Couple months ago I was working on my [personal website](http:/www.cavigneaux.net) that I decided to re-write from scratch since most of the existing things had to be scratched off and replaced.

This site has a really simple purpose: present myself, my knowledges and projects I work on. A way for potential customers & partners to know a bit more about me and my skills.

Rather than writing a full-fledge Rails app I decided to create a simple static website since most of the content will rarely change. Things that need to be fetch dynamically will use Javascript.

# Nanoc

No I was not ready to write all pages with raw HTML / CSS again… [Nanoc](http://nanoc.stoneship.org/) was the best solution for my needs.

Nanoc is a web site generator which can use layouts and compile documents written in [HAML](http://haml-lang.com/) or [Markdown](http://daringfireball.net/projects/markdown/) into multiple directories and static files ready to be push on the web server.

Nanoc is available as a gem and can be installed this way:

{% highlight console %}
gem install nanoc
{% endhighlight %}

Next step is to create website skeleton:

{% highlight console %}
nanoc create_site cavigneaux.net
{% endhighlight %}

A new directory is created and contains:

- a YAML config file
- a `Rules` file which declares available routes and compilation rules
- a `content` directory with an HTML index file and a default stylesheet
- a default layout in `layouts` directory

First thing to do is to compile this skeleton to understand what is happening under the hood:

{% highlight console %}
nanoc compile
{% endhighlight %}

It should create a new directory named `output` containing the static version of the website. We can browse it using the bundle web server:

{% highlight console %}
nanoc view
{% endhighlight %}

This starts a Ruby webserver at [http://localhost:3000](http://localhost:3000).

The default page was generated using files in `content` directory. It's an introduction to Nanoc.

We're ready to create our site adding HTML, Javascript, CSS and images files.

I decided to use HAML for layout and content files, SCSS for stylesheets and CoffeeScript for javascripts.

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

## Nanoc binary

Nanoc is shipped with a binary to create your app skeleton but it also allows you to:

- (auto-)compile your sources into the resulting HTML
- list available plugins
- validates HTML / CSS
- deploy your files to the server
- …

# HAML / SCSS / CoffeeScript

As said earlier, I use HAML to generate both website layout and content.

# Guard

I'm a lazy guy. If I can automate things, I'll do for sure.

So here comes Guard which is a gem that can watch directory and file changes and acts on changes.

I use it to automatically:

- install gems when Gemfile is modified
- re-compile website when config, rules, layout or contents changes
- handle livereload to see modification live (without reloading page by hand) on my browser when a css, a js or an html file has changed
- convert coffeescript files to CSS files

{% highlight ruby %}
guard 'bundler' do
  watch('Gemfile')
end

guard 'nanoc' do
  watch(/^config.yaml/)
  watch(/^Rules/)
  watch(/^layouts\//)
  watch(/^content\//)
end

guard 'livereload' do
  watch(%r{public/.+\.(css|js|html)})
endwr

guard 'coffeescript', :input => 'content/coffeescripts', :output => 'content/javascripts'
{% endhighlight %}

This way I always have the latest version in my browser and can modify anything on the fly.

# Rules

One of the first thing to do is to defined your rules, this is how nanoc knows how it should compile files. Here's the one I use on [cavigneaux.net](http://www.cavigneaux.net) along with some explanations:

{% highlight ruby %}
#!/usr/bin/env ruby

# The order of rules is important: for each item, only the first matching
# rule is applied.

# Nothing special to do with images / javascript / asset
# Just copy it to public directory
compile '/images/*/' do
end

# CoffeeScript file are convert to JS file by my Guardfile

compile '/coffeescripts/*/' do
end

compile '/javascripts/*/' do
end

compile '/assets/*/' do
end


# Stylesheets are written using SCSS so we ask nanoc
# to process them through filters to generate CSS
#
# Rainpress is used to compress resulting CSS files
compile '/stylesheets/*/' do
  filter :sass, :syntax => :scss
  filter :rainpress
end

# Process sitemap.xml through HAML. The HAML file only
# calls a helper to generate sitemap file
compile 'sitemap' do
  filter :haml
end

# Here's a special route for english version of my website
# Root / default is in french
#
# We also process our content through Ruypants filter to have
# nice typographic punctuation
compile '/en/*' do
  filter :haml
  layout 'en'
  filter :rubypants
end

# Let's do the same for main (French) content
compile '*' do
  filter :haml
  layout 'default'
  filter :rubypants
end

# Add some routes for assets, ensure good extension
# is used by Nanoc since it tries to generate HTML
# by default
route '/images/*/' do
  item.identifier.chop + '.' + item[:extension]
end

route '/stylesheets/*/' do
  item.identifier.chop + '.css'
end

# Don't want to be able to access coffescript files
# on the public website
route '/coffeescripts/*' do
  nil
end

route '/javascripts/*/' do
  item.identifier.chop + '.js'
end

route '/assets/*/' do
  item.identifier.chop + '.' + item[:extension]
end

route 'sitemap' do
  item.identifier.chop + '.xml'
end

# Allow auto-serving of index.html file
# on URLs like /posts/
route '*' do
  item.identifier + 'index.html'
end

# Use HAML to process our layout
# We want HTML5 doctype
layout '*', :haml, :format => :html5
{% endhighlight %}

# Include helpers

You may need to load extra-functionalities of Nanoc or create your own one. The lib/ directory is a good one for this since it is auto-loaded by Nanoc. This is the perfect place to put some kind of initializers and include helpers. Here is mine:

{% highlight ruby %}
include Nanoc3::Helpers::LinkTo
include Nanoc3::Helpers::XMLSitemap
{% endhighlight %}

Pretty straightforward, I only need some link_to helper to use links creation. I also want to add a sitemap to my website. This modules will be of help.

# Compile and deploy your website

In my case I use the included [Rsync](http://rsync.samba.org) solution to upload my website to the server. With the sample configuration I gave you at the beginning, I tell Nanoc to use Rsync to keep my file in sync through SSH. I also asked Rsync to delete files not used anymore on the server.

# Conclusion

With Nanoc you can have a powerful and customizable website content manager up and running in less than an hour. You'll have to do the config stuff once and then you only have to think about your content, write it then deploy with a single command.

Nanoc is really worth checking it if you need a simple yet powerful content managing system for a static website.

You can go further and learn some more tricks by reading my [website source code](https://bitbucket.org/Bounga/cavigneaux.net).
