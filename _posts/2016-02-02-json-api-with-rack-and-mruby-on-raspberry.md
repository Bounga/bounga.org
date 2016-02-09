---
layout: post
title: Running a JSON API with Rack and mruby on a Raspberry Pi
category: raspberry
tags: ['ruby', 'rack', 'API']
---

I recently got a [Raspberry Pi 2](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/) so I wanted to experiment with home automation, surveillance system, movement detection and stuff like that.

Thinking of the things I could do with it I quickly realized that I would need an [API](https://en.wikipedia.org/wiki/Application_programming_interface) at home to centralized everything. I immediately thought it was a work for a Raspberry!

It would be cool if I could write an API using Ruby on a Raspberry. The thing is Ruby is slow and Raspberry has modest hardware so it may be difficult to run a Ruby web app.

Why not run a benchmark of a simple example to get an overview of what this tiny thing can handle. That's what I've done.

## Running Ruby through mruby ##

First thing first. What to use as a webserver to run my Ruby app?

The obvious answer is to use something based on [mruby](https://mruby.org) which is a lightweight implementation of the Ruby language. mruby can be embed by any other program, let's say a webserver which will allow for Ruby to be interpreted on it.

This brings me to my second point. Is there any webserver out there which can run Ruby through mruby? Yes, there's one called [H2O](https://h2o.examp1e.net) which is specifically crafted to be fast.

It's a young project but it's evolving fast. I'm pretty sure we'll hear about it more and more in the next months.

### Installing H2O on the Raspberry ###

Right now the only way to get it on the Raspberry with mruby enabled is to compile it. So let's go.

First we'll clone H2O repository locally, install dependencies then build it:

{% highlight bash %}
$ git clone https://github.com/h2o/h2o.git
$ cd h2o
$ sudo apt-get install cmake bison
$ cmake -DCMAKE_C_FLAGS_RELEASE= \
    -DCMAKE_CXX_FLAGS_RELEASE= \
    -DCMAKE_INSTALL_PREFIX=./dist \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_FIND_FRAMEWORK=LAST \
    -DCMAKE_VERBOSE_MAKEFILE=ON \
    -DWITH_MRUBY=ON . \
    -Wno-dev
$ make
$ make install
{% endhighlight %}

Now that H2O is installed, it's time to write our API!

## Writing Rack Application

For the sake of tests I only wrote a really simple API app. That's not a real world app but a good starting point to validate the concept.

We're going to write a Rack application that reads a key from a Hash in memory and return it as JSON payload. It could be possible to read from [Redis](http://redis.io) or from a database but that's not the point here.

So here is our Rack app:

`json_api.rb`

{% highlight ruby %}
class JsonApi
  def initialize
    @storage = {foo: "bar"}
  end

  def call(env)
    body = JSON.generate({foo: @storage[:foo]})

    [
      200,
        {'Content-Type'   => 'application/json',
         'Content-Length' => body.size},
       [body]
     ]
  end
end

JsonApi.new
{% endhighlight %}

Really simple as I told before. On initialization, it creates an instance variable which will hold some data.

On `call` (remember this is a Rack app) it will generate a JSON from the value available in the instance variable and use it as the response body.

We then create the standard response structure needed for a Rack app. An array with the response code, headers and body.

Last thing we have to do is to instantiate our class.

## Configuring H2O

And now the final step to have a working example. The missing step is to configure H2O to serve our Rack app. We need to tell it where to listen and what to serve:

`h2o.conf`

{% highlight json %}
listen: 8080
hosts:
  "localhost:8080":
    paths:
      /json_api:
        file.dir: .
        mruby.handler-file: json_api.rb
    access-log: /dev/null
{% endhighlight %}

H2O is going to listen on 8080 port. When a request goes to the `/json_api` path, H2O will search for a file named `json_api.rb` in the same directory as the configuration file. This file has to be a Rack app.

We don't want to keep track of logs so we write them in `/dev/null` which is some kind of black hole.

We can now start the server:

{% highlight bash %}
$ dist/bin/h2o --conf h2o.conf
{% endhighlight %}

and test it from another computer:

{% highlight bash %}
➜ curl http://raspberry_ip:8080/json_api/
{"foo":"bar"}
{% endhighlight %}

It works!

## Benchmark ##

Now that we have all we needed, get back to our main intent, benchmarking our Raspberry Pi's ability to act as an API server.

To bench our setup we'll use [wrk](https://github.com/wg/wrk) which is a modern HTTP benchmarking tool capable of generating significant load when run on a single multi-core CPU.

So from my MacBook connected on the local network using WiFi, I'm going to stress the API which is on the Raspberry. The Raspberry is also connected to the local network using a WiFi dongle.

Let's go for the bench:

{% highlight bash %}
$ wrk --threads 2 --duration 10 http://192.168.1.14:8080/

Running 10s test @ http://192.168.1.14:8080/
  2 threads and 10 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    12.21ms    4.97ms  59.19ms   79.62%
    Req/Sec   413.08     69.50   555.00     71.00%
  8262 requests in 10.05s, 1.47MB read
Requests/sec:    821.71
Transfer/sec:    149.26KB
{% endhighlight %}

## Conclusion ##

**821 requests/sec** seems really good to me for a Ruby app on such a modest hardware. I have no doubt that it can easily be used as a personnal API server to handle every single need, even under heavy use. I'm now sure that even if all my stuff in the house become connected it will scale!

To be honest it can even be used for a small public API.
