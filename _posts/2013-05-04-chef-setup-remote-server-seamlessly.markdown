---
layout: post
title: Chef - Setup remote server seamlessly
category: unix
tags: ['unix', 'system', 'admin', 'automation']
---

[Chef](http://wiki.opscode.com/display/chef/Home) is a [Ruby Gem](https://rubygems.org/gems/chef) which aims to help sys admins in the process of **managing servers**.

Chef brings the ability to write ruby code rather than running commands on the servers to manage them. You'll be able to install softwares, create users, write configuration files and so on through code.

This is a very nice tool to manage your servers in a predictable, reproducible and evolving way without the headache of knowing what is installed on which server and things like that.

If you need to setup more than one server to be exact clones, Chef is your man! It can help setting up staging / production servers or clusters. All you have to do is to write recipes once and then you're ready to install, restore, clone servers in a single command.

Let's see the basics of this systems integration framework.

# Core concept

Chef runs a bunch of recipes to set up your servers. You can run your recipes several times and Chef will guess what has already been done or what changed so you don't have to care about this.

This means that if you've already installed softwares (eg: a web server, a SMTP, …) and you run your Chef recipes it will not try to install it again or change anything in the configuration file.

If you want to upgrade the web server version, all you have to do is to set the version in your recipe and then run Chef again. Chef will detect if the installed version and the recipe one are not the same. If they are different it will install the one specified in the recipe.

Chef recipes are organized in a standard directory structure. It provides many useful tools out of the box to automate tasks. You can do anything through Chef so you can really do whatever you want as if you were working interactively on the server via SSH. Install a package using your system packing tool, write configuration files, create a user or a directory tree, set up a backup strategy. To sum up, you can do whatever you can think of on the server.

You can create your own recipes to match your needs or use existing recipes. You have to be careful with existing recipes you borrowed from somewhere 'cause you leave the door open to your servers when you execute Chef recipes. Be sure to check that you're not using nasty recipes.

To conclude on Chef concept you can see it as test suite for your servers. It checks things on the servers to see if it matches your expectations (recipes). If it doesn't Chef will do the required operations to match your expectations.

# Why Chef?

I used to write my own shell / Ruby scripts to automate some sequences of commands on servers I had to install and maintain.

It became tedious pretty quickly 'cause it doesn't handle version management and things like that out of the box. Moreover it's error-prone to write this kind of code from scratch and it's hard to ensure that it will be portable across all the servers / Unix flavors.

Chef is **the way** to go to have readable and maintainable code because it's shipped with all the tools needed to efficiently write your installation / configuration process even if it's a complex one.

Once your recipes are ready and work on a given server, you can be sure that you will be able to repeat this process on any other server even if the Unix distribution is not the same. If you're not a sys-admin this is a huge advantage which will make you comfortable with deploying a new server.

Chef is also a developer best friend since it allows to install the same OS, softwares and services as on production server using a virtual machine with only one command. That's a great way to ensure the app acts as it will in production environment.

# Chef's directory structure

Chef has a specific directory organization and terms to define things. Let's see what the major concepts of Chef are:

- `node` is a host (db server, web server, mail server, …) where Chef client will run
- `Chef server` is a web server that stores info about your production servers
- `Chef client` is the program which actually connect to a node to do the job
- `Chef solo` is a Chef client that doesn't rely on Chef server for configuration
- `recipe` contains commands to run on a node
- `resource` is the basic unit used by recipes (files, directory, users, …)
- `cookbook` is a collection of recipes
- `role` is configuration specific to given servers
- `run list` is a list of recipes and roles to run
- `attribute` is a variable usable in recipes and templates

# Discovering Chef solo

The most simple / straightforward usage of Chef is to use `chef-solo` which doesn't rely on Chef server so it's easier to start with it.

Let's see how to use it and what we can do.

## Setup the server(s)

Servers that will be handled using Chef need minimal requirements. Chef relies on Ruby so we need to install Ruby and Chef above all else.

My favorite Unix flavor is Debian so that's what I'm gonna use here. My examples will work with any flavor but you need to use your distribution package manager to setup your server.

First you have to ssh into server(s) you want set up:

{% highlight console %}
$ ssh root@my_server
{% endhighlight %}

Then you need to install Ruby to get Chef running:

{% highlight console %}
# apt-get -y update
# apt-get -y install ruby1.9.1-full
# gem install chef ruby-shadow --no-ri --no-rdoc
{% endhighlight %}

Note you can still install [rbenv](https://github.com/sstephenson/rbenv) afterward using Chef then play with Ruby versions as usual. IMO you should use Chef as much as possible when you decide to use it for your server configuration.

Let's check that Ruby and Chef are available:

{% highlight console %}
# ruby -v
# chef-solo -v
{% endhighlight %}

There's only one task remained on the server. We need to create the root directory required by Chef for its cookbooks:

{% highlight console %}
# mkdir /var/chef
{% endhighlight %}

Now you can go back to your local machine where you'll write the cookbook.

## Basic recipes

You can, locally, create your main cookbook directory skeleton:

{% highlight console %}
$ mkdir -p cookbooks/main/recipes
{% endhighlight %}

Note that I created only one `main` cookbook where we will do all our testing stuff but Chef users like to have one cookbook per software to install or task to be done.

Open your editor of choice and create the chef-solo configuration file which specifies where to find the cookbooks and variables file:

`cookbooks/solo.rb`

{% highlight ruby %}
cookbook_path File.expand("../cookbooks", __FILE__)
json_attribs File.expand("../node.json", __FILE__)
{% endhighlight %}

Next create the default recipe:

`cookbooks/main/recipes/default.rb`

{% highlight ruby %}
package "git-core"
package "zsh"
{% endhighlight %}

These two lines tell our recipe to install `git` and `zsh`. This will use platform dependent installation that is to say `apt-get` for Debian.

Then you need to create a run list to define which recipes you want to run:

`cookbooks/node.json`

{% highlight js %}
{
  "run_list": ["recipe[main]"]
}
{% endhighlight %}

We have our minimalistic version of our cookbook, we're now able to sync it on the target server(s) and run it. I like to copy my SSH key on the server so I don't have to type in my password for every single command sent.

To copy this key, I use `ssh-copy-id`. I installed it using [homebrew](http://mxcl.github.io/homebrew/):

{% highlight console %}
$ brew install ssh-copy-id
{% endhighlight %}

then

{% highlight console %}
$ ssh-copy-id root@my_server
$ rsync -r . root@my_server:/var/chef
$ ssh root@my_server "chef-solo -c /var/chef/solo.rb"
{% endhighlight %}

You should see that `git-core` and `zsh` packages were installed. If you run chef-solo command again it first check current state and doesn't try to install anything.

That's it for the very basic stuff.

Chef offers some useful [default cookbooks](https://github.com/opscode-cookbooks) we're going to use next to enhance our deployment strategy.

### Updates to cookbooks

If you update something in the recipes and run the chef-solo command again, all differences with current server state will be taken into account. Now if your Chef cookbooks are tracked using a [SCM](http://en.wikipedia.org/wiki/Revision_control) you can easily go back and forth in your server configurations.

## Add users to servers

First thing you definitively want to do on your server is to create a unprivileged user to use for common tasks.

Chef comes with [User resource](http://wiki.opscode.com/display/chef/Resources#Resources-User) to ease user creation and modification on the server.

So our user will need a password shadow hash we can create this way:

{% highlight console %}
$ openssl passwd -1 "my_user_awesome_password"
{% endhighlight %}

Then you can edit your `cookbooks/main/recipes/default.rb` file to add:

{% highlight ruby %}
user "bounga" do
  comment "Deploiment account"
  gid "users"
  home "/home/bounga"
  shell "/bin/zsh"
  password "$1$JJsvHslV$szsCjVEroftprNn4JHtDi."
  supports manage_home: true
end
{% endhighlight %}

As you can see, you have fine-grained tuning for user handling and all the other resources available are as tunable. That's a lot of comfort compared to a shell-script you'll write by yourself and you'll need to maintain over the time.

Now we need to sync the cookbook on the server and run chef-solo again:

{% highlight console %}
$ rsync  -r . root@my_server:/var/chef
$ ssh root@my_server "chef-solo -c /var/chef/solo.rb"
{% endhighlight %}

We have a very simple cookbook for now but if you want something more complex I really encourage you to keep all the hardcoded values such as usernames, passwords and so on in a dedicated file.

You can use the existing `cookbooks/node.json` file to store all your variables:

`cookbooks/node.json`

{% highlight js %}
{
  "user": {
    "name": "bounga",
    "password": "$1$JJsvHslV$szsCjVEroftprNn4JHtDi."
  },
  "run_list": ["recipe[main]"]
}
{% endhighlight %}

Now you have user informations stored in the json file, you still need to use it in recipes:

`cookbooks/main/recipes/default.rb`

{% highlight ruby %}
user node[:user][:name] do
  comment "#{node[:user][:name]} User"
  gid "users"
  home "/home/#{node[:user][:name]}"
  shell "/bin/zsh"
  password node[:user][:password]
  supports manage_home: true
end
{% endhighlight %}

## Use templates to create remote files

You'll often need to create default configuration files on your server. Here come [templates](http://wiki.opscode.com/display/chef/Resources#Resources-Template) which allow you to create a skeleton for files and deploy it on the server.

Create a directory called `coobooks/main/templates/default` and add this to the bottom of `coobooks/main/recipes/default.rb`:

{% highlight ruby %}
template "/home/#{node[:user][:name]}/.zshrc" do
  source "zshrc.erb"
  owner node[:user][:name]
end
{% endhighlight %}

The goal is to create a default .zshrc file for our user. The last step is to create the template file:

`coobooks/main/templates/default/zshrc.erb`

{% highlight erb %}
alias l='ls -lA1'
{% endhighlight %}

Don't forget you're in an erb file so you can use variables defined in node.json and generate content using Ruby code.

## Define ronjobs

Another recurrent need on remote servers is to [create cronjob](http://wiki.opscode.com/display/chef/Resources#Resources-Cron) for automated tasks:

Add this code to the bottom of `coobooks/main/recipes/default.rb`:

{% highlight ruby %}
cron "noop" do
  hour "5"
  minute "0"
  command "/bin/true"
end
{% endhighlight %}

Sync your cookbook on the server and run chef-solo and a new cronjob will be added. This cronjob will run the `/bin/true` command at 5 every morning. You can also add some conditions for the cronjob to execute.

# You still don't know the real power of Chef

I can't go through all Chef's features in this post but be sure to check the [documentation](http://wiki.opscode.com/display/chef/Documentation) which contains a load of info.

You can for example discover how to handle services, use git, install gems, manage files, directories, users, groups and logs in an efficient and clean way.

The [community cookbooks](http://community.opscode.com) are a nice way to discover available cookbooks for databases, web-servers, monitoring and more. Download the cookbook you need, let's say "[nginx](http://community.opscode.com/cookbooks/nginx)", copy it in your Chef coobooks directory then you can use it :

Add this to the bottom of `coobooks/main/recipes/default.rb`:

{% highlight ruby %}
package "nginx"
{% endhighlight %}

You can and should explore cookbooks you download to understand what they can do and how you can structure your own cookbooks.

With this community cookbooks site, you can have a server up and running in a couple minutes. Adding a new clone server will be faster than light and updates are as simple as changing your recipe and executing a single command for each of your servers.

# Chef, a developer's best friend

If you're a developer who likes to spend time writing code rather than doing sys-admin, I'm pretty sure you'll love Chef.

I hope this quick introduction to Chef will be helpful for those of you who want to put servers up & running quickly in a repeatable manner.

I've barely scratched the surface and there's a **lot** **lot** more to know about Chef. It's a real server provisioning tool that can do anything on the server for you through Ruby code. Don't be afraid to write your own cookbooks with custom resources to fit your needs.
