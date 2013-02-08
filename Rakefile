require 'rubygems'
require 'jekyll'
require 'kramdown'
include Jekyll::Filters

HOST = "www.bounga.org"
REMOTE_USER = "synbioz"
REMOTE_DIRECTORY = "/var/www/bounga.org/"

task :default => [:build]

desc 'Generate category pages'
task :categories do
  puts "Generating category pages..."

  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read_posts('')

  FileUtils.mkdir_p("categories")

  site.categories.each do |category, posts|
    unless File.exists?("categories/#{category}.html")

      html = ''
      html << <<-HTML
---
layout: default
title: Posts under "#{category}" category
---
<section id="#{category}">
  <h2>{{ page.title }}</h2>
  <ul>
  {% for post in site.categories['#{category}'] %}
    <li>{{ post.date | date_to_string }} - <a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
</section>
HTML

      File.open("categories/#{category}.html", 'w') do |file|
        file.puts html
      end
    end
  end

  puts 'Category pages generation done.'
end

desc 'Generate tags cloud'
task :tags do
  puts "Generating tags cloud..."

  options = Jekyll.configuration({})
  site = Jekyll::Site.new(options)
  site.read_posts('')


  FileUtils.mkdir_p("tags")

  tagcloud = ''

  site.tags.each do |label, posts|
    unless File.exists?("tags/#{label}.html")
      html = ''
      html << <<-HTML
---
layout: default
title: Posts tagged "#{label}"
---
<section id="#{label}">
  <h2>{{ page.title }}</h2>
  <ul>
  {% for post in site.tags['#{label}'] %}
    <li>{{ post.date | date_to_string }} - <a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
</section>
HTML

      File.open("tags/#{label}.html", 'w') do |file|
        file.puts html
      end
    end

    ratio = ((posts.length.to_f / site.posts.length.to_f) * 100).to_i
    ratio = ratio + (10 - ratio % 10)
    tagcloud += "<li><a href='/tags/#{label}' class='tag#{ratio}'>#{label}</a></li> "
  end

  File.open("_includes/tags.html", 'w') do |file|
    file.puts tagcloud
  end

  puts 'Tags cloud generation done.'
end

desc 'Build public content'
task :build => [:categories, :tags] do
  system('jekyll --no-auto')
end

desc 'Deploy site on web server'
task :deploy => [:build] do
  puts "Sync files on Web server"
  system("rsync -avz --delete -e ssh public/ #{REMOTE_USER}@#{HOST}:#{REMOTE_DIRECTORY}")
  puts "Deployment done!"
end