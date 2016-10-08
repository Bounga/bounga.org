---
layout: post
title: Using Sidekiq with Hanami
category: Ruby
tags: ['ruby', 'hanami', 'sidekiq']
---

One of the gems I often use in my [Ruby on Rails](http://rubyonrails.org/)
projects is [Sikekiq](http://sidekiq.org/). For those of you who
don't know about it, it allows to easily manage background
processes.

As I'm now using [Hanami](http://hanamirb.org/) more and more for my
personal projects it quickly became essential to me to integrate
it in Hanami. Fortunately Sidekiq is framework agnostic and easy
to get up and running in any Ruby project.

## Setting up Sidekiq ##

As in every other Ruby project, the first thing to do is to add
Sidekiq dependency in your `Gemfile` then `bundle install`:

{% highlight ruby %}
gem 'sidekiq'
{% endhighlight %}

Once installed we need to configure it, it's pretty straightforward
since the only thing we need to define is the Redis URL to use.

We're going to encapsulate this configuration in its own file
`config/sidekiq.rb`:

{% highlight ruby %}
Sidekiq.configure_server do |config|
  config.redis = { url: ENV.fetch('REDIS_URL') }
end

Sidekiq.configure_client do |config|
  config.redis = { url: ENV.fetch('REDIS_URL') }
end
{% endhighlight %}

There are two things here. First we make use of environment variable
to define and get Redis URL. It is a best practice pushed by Hanami
which already setup all you need to use environment variable
effectively. This avoid hard-coding it and ease
configuration for deployment tools.

The other noticeable thing is that we use `fetch` to retrieve the
value associated to our `REDIS_URL` key rather than accessing it
through `[]`. The advantage with `fetch` is it will raise an `IndexError`
exception if the key doesn't exist so we'll notice the missing key /
value pair as soon as we start the server.

We then need to tell Hanami that it should load it on start. We could
have put this file in something like `apps/web/config/initializers`
and it would have been loaded automatically without any further
configuration but I don't think that Sidekiq is tied to one of our
app, it's more of a transverse component that can be used by any of
our inner apps.

So let's instruct our Hanami app about this file to load by editing
`config/environment.rb`:

{% highlight ruby %}
require_relative './sidekiq'
{% endhighlight %}

We can now set Redis URL in our `.env.development` file which will be
sourced automatically on server start.

## Scheduling tasks from our application  ##

Since our workers are not tied to a given app, we're going to put them
in `lib/<application_name>/workers` because `lib` directory is
dedicated to business logic and domain model.

So let's say we create a worker that does an heavy processing, it
could be something like
`lib/<application_name>/workers/heavy_processing.rb`:

{% highlight ruby %}
class HeavyProcessing
  include Sidekiq::Worker

  def perform(some, params)
    # Do some heavy processing
  end
end
{% endhighlight %}

As you can see it's just plain Sidekiq code with a class that
`include`s `Sidekiq::Worker` and defines a `perform` method that can
take any arguments.

Now you can use it from anywhere in your code to schedule background
jobs. In our example the call would be something like:

{% highlight ruby %}
HeavyProcessing.perform_async(:something, {hard: "to process"})
{% endhighlight %}

Ok the task is now scheduled but we still need to run Sidekiq deamon
to watch for those scheduled tasks and run them.

## Running Sidekiq deamon along with Hanami ##

The first dumbest way to do this is to run your Hanami app from a
terminal:

{% highlight terminal %}
$ bin/hanami server
{% endhighlight %}

then open up another tab and run Sidekiq manually:

{% highlight terminal %}
$ bin/sidekiq -e development -r ./config/environment.rb
{% endhighlight %}

It's pretty tedious and your workmates won't thanks you for that.

Another simpler, more reliable way to do this is to use a `Procfile`. It's
a way to define what must be started and how it should be started. You
can then use something
like [Foreman](https://github.com/ddollar/foreman) to read this
`Procfile` and do the job for you:

{% highlight text %}
app: bin/hanami server
sidekiq: bin/sidekiq -e development -r ./config/environment.rb
{% endhighlight %}

You're now able to start every piece of software needed to run the app
with a single command:

{% highlight terminal %}
$ foreman start
{% endhighlight %}

If you ever had some more dependencies, you'll just have to add it to
your `Procfile`.

For [Docker](https://www.docker.com/technologies/overview)'s fan you
can also use it
with [docker-compose](https://docs.docker.com/compose/overview/) to
start all your services.

## Let's schedule ##

This is a pretty long post for such a simple thing to setup but I
wanted to be very didactic so even the new comers to Hanami and
Sidekiq can understand what happens.

Now you're ready to schedule background jobs from your Hanami app!
Make good use of it.
