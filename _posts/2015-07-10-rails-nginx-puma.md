---
layout: post
title: Setting up Rails, Nginx and Puma
category: unix
tags: ['unix', 'system', 'admin', 'hosting']
---

When [Rails 4](http://edgeguides.rubyonrails.org/4_0_release_notes.html) was released it offered a bunch new features and deprecations :

[![Rails 4 new features and deprecations](http://guides.rubyonrails.org/images/rails4_features.png "Rails 4 new features and deprecations")](http://guides.rubyonrails.org/images/rails4_features.png)

You can see that thread safety is since on by default so a Rack server can safely create threads for your Rails app rather than forking process and waste cpu cycles and memory.

If you didn't switch to a Rack compatible server that can efficiently makes use of threads then it's time to do so. I decided to switch from [Unicorn](https://github.com/defunkt/unicorn) (process forks) to [Puma](http://puma.io) which is thread oriented.

You'll see a great memory improvement if you switch and you'll be able to use
threads effectively in your app if you want to since Rails is now fully thread-safe.

Here are the three steps to have a [Nginx](http://nginx.org) / [Puma](http://puma.io)  / [Capistrano](http://capistranorb.com) stack up & running for a Rails app.

# Configuring Nginx

## Common config

You probably already have this setted up if you were using Nginx / Unicorn stack but here is a quick reminder for Nginx global configuration:

`/etc/nginx/nginx.conf`

{% highlight nginx %}
user www-data;
worker_processes 8;
pid /var/run/nginx.pid;

events {
  worker_connections 1024;
  multi_accept on;
}

http {
  ##
  # Basic Settings
  ##

  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 5;
  types_hash_max_size 2048;
  server_names_hash_max_size 512;
  server_names_hash_bucket_size 128;

  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  ##
  # Logging Settings
  ##

  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;

  ##
  # Gzip Settings
  ##

  gzip on;
  gzip_disable "msie6";
  gzip_http_version 1.1;
  gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

  ##
  # Virtual Host Configs
  ##

  include /etc/nginx/conf.d/*.conf;
  include /etc/nginx/sites-enabled/*;
}
{% endhighlight %}


Pretty straigth forward, nothing new here.

## Vhost file

For example `/etc/nginx/sites-enabled/01-bounga`

{% highlight nginx %}

# Upstream server (Puma socket)
upstream bounga {
  server unix:/var/www/bounga/shared/sockets/puma.sock;
}

# Redirect non-www requests to www host
server {
  server_name bounga.org;
  return 301 $scheme://www.bounga.org$request_uri;
}

# Config for vhost
server {
  listen 80;
  server_name www.bounga.org;

  access_log /var/log/nginx/bounga.access.log;
  error_log /var/log/nginx/bounga.error.log;

  root /var/www/bounga/current/public;

  # direct to maintenance if this file exists
  if (-f $document_root/system/maintenance.html) {
    rewrite  ^(.*)$  /system/maintenance.html last;
    break;
  }

  # assets caching
  location ~ ^/(assets)/ {
    access_log  off;
    gzip_static on;
    expires     max;
    add_header  Cache-Control public;
    add_header  Last-Modified "";
    add_header  ETag "";

    open_file_cache          max=1000 inactive=500s;
    open_file_cache_valid    600s;
    open_file_cache_errors   on;
    break;
  }

  # serve static file directly
  try_files $uri/index.html $uri @bounga;

  # App proxying
  location @bounga {
    proxy_redirect    off;
    proxy_set_header  Host $host;
    proxy_set_header  X-Real-IP $remote_addr;
    proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass        http://bounga;
  }

  error_page 500 502 503 504 /500.html;
  keepalive_timeout 10;

  location = /favicon.ico {
    expires    max;
    add_header Cache-Control public;
  }
}
{% endhighlight %}

Once again nothing new here. You may have to change some values or add some directives depending on your app behaviour but this config example should work in most usecases.

# Configuring Puma

This is were you'll learn new things. Puma default settings are safe so you can use it for your Rails apps without even creating a config file. It will work just fine.

Nonetheless maybe you'll need to tweak your config so here are some insigths to be able to tune Puma behaviour.

`config/puma.rb`:

{% highlight ruby %}
#!/usr/bin/env puma

# The directory to operate out of.
#
# The default is the current directory.
#
# directory '/u/apps/lolcat'

# Use an object or block as the rack application. This allows the
# config file to be the application itself.
#
# app do |env|
#   puts env
#
#   body = 'Hello, World!'
#
#   [200, { 'Content-Type' => 'text/plain', 'Content-Length' => body.length.to_s }, [body]]
# end

# Load "path" as a rackup file.
#
# The default is "config.ru".
#
# rackup '/u/apps/lolcat/config.ru'

# Set the environment in which the rack's app will run. The value must be a string.
#
# The default is "development".
#
# environment 'production'

# Daemonize the server into the background. Highly suggest that
# this be combined with "pidfile" and "stdout_redirect".
#
# The default is "false".
#
# daemonize
# daemonize false

# Store the pid of the server in the file at "path".
#
# pidfile '/u/apps/lolcat/tmp/pids/puma.pid'

# Use "path" as the file to store the server info state. This is
# used by "pumactl" to query and control the server.
#
# state_path '/u/apps/lolcat/tmp/pids/puma.state'

# Redirect STDOUT and STDERR to files specified. The 3rd parameter
# ("append") specifies whether the output is appended, the default is
# "false".
#
# stdout_redirect '/u/apps/lolcat/log/stdout', '/u/apps/lolcat/log/stderr'
# stdout_redirect '/u/apps/lolcat/log/stdout', '/u/apps/lolcat/log/stderr', true

# Disable request logging.
#
# The default is "false".
#
# quiet

# Configure "min" to be the minimum number of threads to use to answer
# requests and "max" the maximum.
#
# The default is "0, 16".
#
# threads 0, 16

# Bind the server to "url". "tcp://", "unix://" and "ssl://" are the only
# accepted protocols.
#
# The default is "tcp://0.0.0.0:9292".
#
# bind 'tcp://0.0.0.0:9292'
# bind 'unix:///var/run/puma.sock'
# bind 'unix:///var/run/puma.sock?umask=0111'
# bind 'ssl://127.0.0.1:9292?key=path_to_key&cert=path_to_cert'

# Instead of "bind 'ssl://127.0.0.1:9292?key=path_to_key&cert=path_to_cert'" you
# can also use the "ssl_bind" option.
#
# ssl_bind '127.0.0.1', '9292', { key: path_to_key, cert: path_to_cert }

# Code to run before doing a restart. This code should
# close log files, database connections, etc.
#
# This can be called multiple times to add code each time.
#
# on_restart do
#   puts 'On restart...'
# end

# Command to use to restart puma. This should be just how to
# load puma itself (ie. 'ruby -Ilib bin/puma'), not the arguments
# to puma, as those are the same as the original process.
#
# restart_command '/u/app/lolcat/bin/restart_puma'

# === Cluster mode ===

# How many worker processes to run.
#
# The default is "0".
#
# workers 2

# Code to run when a worker boots to setup the process before booting
# the app.
#
# This can be called multiple times to add hooks.
#
# on_worker_boot do
#   puts 'On worker boot...'
# end

# Code to run when a worker boots to setup the process after booting
# the app.
#
# This can be called multiple times to add hooks.
#
# after_worker_boot do
#   puts 'After worker boot...'
# end

# Code to run when a worker shutdown.
#
#
# on_worker_shutdown do
#   puts 'On worker shutdown...'
# end

# Allow workers to reload bundler context when master process is issued
# a USR1 signal. This allows proper reloading of gems while the master
# is preserved across a phased-restart. (incompatible with preload_app)
# (off by default)

# prune_bundler

# Preload the application before starting the workers; this conflicts with
# phased restart feature. (off by default)

# preload_app!

# Additional text to display in process listing
#
# tag 'app name'

# Change the default timeout of workers
#
# worker_timeout 60

# === Puma control rack application ===

# Start the puma control rack application on "url". This application can
# be communicated with to control the main server. Additionally, you can
# provide an authentication token, so all requests to the control server
# will need to include that token as a query parameter. This allows for
# simple authentication.
#
# Check out https://github.com/puma/puma/blob/master/lib/puma/app/status.rb
# to see what the app has available.
#
# activate_control_app 'unix:///var/run/pumactl.sock'
# activate_control_app 'unix:///var/run/pumactl.sock', { auth_token: '12345' }
# activate_control_app 'unix:///var/run/pumactl.sock', { no_token: true }
{% endhighlight %}

# Configuring Capistrano

Now we need to setup Capistrano to be able to handle Puma on deploys.

First you'll need to add a gem to your `Gemfile`:

{% highlight ruby %}
gem 'capistrano3-puma'
{% endhighlight %}

Then in your `Capfile` you'll have to require it:

{% highlight ruby %}
require 'capistrano/puma'
{% endhighlight %}

It will add some tasks to you Capistrano tasks:

{% highlight console %}
$ bundle exec cap puma:start
$ bundle exec cap puma:restart
$ bundle exec cap puma:stop
$ bundle exec cap puma:phased_restart
{% endhighlight %}

Now we're done! Everything else is handled for us. We can [tweak the default](https://github.com/seuros/capistrano-puma/blob/master/lib/capistrano/tasks/puma.cap#L3) behaviour but in most cases it will work as expected, for example `bundle exec cap deploy` will [restart Puma for us](https://github.com/seuros/capistrano-puma/blob/master/lib/capistrano/tasks/puma.cap#L172) at the end of the deploy.

# Puma, a developer's best friend

Puma seems to be the way to go these days if you want to use a proxy and a Rack server. It allows fast and effecient concurrency. Trust me you won't regret this choice.
