---
title: Scheduling tasks without external dependencies in Elixir
category: Elixir
tags: ['elixir', 'OTP']
header:
  overlay_image: /assets/images/headers/elixir.jpg
  teaser: /assets/images/teasers/schedule_in_elixir.jpg
excerpt: "[Elixir](https://elixir-lang.org/) and
[Erlang](https://www.erlang.org/) comes with so much tooling that most
of the time you'll be able to do what you need to without having to
depend on any external software."
---

When you're creating a web app you'll often need to schedule tasks to
be executed in the background.

It's useful to send massive quantity of emails, fetch and store info
from a remote API on regular basis, resize images, batch import large
files, update a knowledge database, cleanup things and so on.

If you're coming from [Ruby](https://www.ruby-lang.org/en/) like me
you're maybe used to relying on tools like
[Sidekiq](https://github.com/mperham/sidekiq),
[Delayed::Job](https://github.com/collectiveidea/delayed_job/),
[Resque](https://github.com/resque/resque) or something similar to
asynchronously execute tasks in the background.

All of the gems listed above are really great solutions for the Ruby
eco-system but require external dependencies.

Using [Elixir](https://elixir-lang.org/) and
[OTP](http://erlang.org/doc/system_architecture_intro/sys_arch_intro.html)
you have all you need at your disposal to handle background and
scheduled tasks. [Erlang](https://elixir-lang.org/) was designed with
this need in mind at core level.

# Use OTP to schedule tasks #

It's really going to be a piece of cake.

We'll do this with zero external dependencies (no database, no Redis,
no nothing) and it will take only a few lines of code to do the job.

Here is the module responsible for scheduling a task:

```elixir
defmodule MyApp.Scheduler do
  use GenServer

  def start_link(_args) do
    GenServer.start_link(__MODULE__, %{})
  end

  def init(state) do
    schedule_work()
    
    {:ok, state}
  end

  def handle_info(:work, state) do
    MyApp.SomeModule.do_some_long_job()
    schedule_work()
    
    {:noreply, state}
  end

  defp schedule_work() do
    Process.send_after(self(), :work, 2 * 60 * 60 * 1000)
  end
end
```

This module uses `GenServer` behavior that will do all the
heavy-lifting for us. It can now keep state, execute code
asynchronously and so on.

Let's dig into it function by function.

## Initialize the server ##

There's one mandatory function that brings our module to life, the
`init/1` function. Its purpose is to initialize our server state.

```elixir
def init(state) do
  schedule_work()
    
  {:ok, state}
end
```

In our example we don't need to maintain a state so we return `{:ok,
state}` without changing anything but before doing so we call our
private function `schedule_work/0` to start our first scheduling count
down. We'll take a closer look at it in a minute.

## Start the server ##

The `start_link/1` function encapsulate the logic responsible for
starting our `GenServer`.

```elixir
def start_link(_args) do
  GenServer.start_link(__MODULE__, %{})
end
```

It isn't required but it's a common pattern to wrap the server start
logic inside a function in your module so the end-user don't have to
understand the underlying logic.

`start_link/1` is really simple, it only calls
`GenServer.start_link/3` with current module as the first argument and
an empty map as the second one. This will call our `init/1` function
under the hood.

In our example, `init/1` will receive `%{}` as its parameter. We don't
really care since we're not going to use it.

The two functions left represent the heart of our module.

## Handle asynchronous calls to our module ##

One of the most notable behavior of the `GenServer` is its ability to
communicate synchronously and asynchronously. It's basically a
"receive" loop which is waiting for messages to handle.

`handle_info/2` is one of the available callbacks, the most generic
one. 

```elixir
def handle_info(:work, state) do
  MyApp.SomeModule.do_some_long_job()
  schedule_work()
    
  {:noreply, state}
end
```

In our example we ask our `handle_info/2` function to pattern match
the `:work` message. When such an message is sent to our module this
function catches it and does its job.

Its job is to do whatever the scheduled task is supposed to do. For
the sake of the example we'll say that the business logic to
be accomplished is handled by another module function
`MyApp.SomeModule.do_some_long_job/0`.

Then next round of work is then scheduled by calling `schedule_work/0`
again.

`handle_info/2` ends by returning a tuple with an unchanged state.

## Schedule the task ##

The last function is a private one, it is not meant to be called
from outside of our `MyApp.Scheduler` module.

```elixir
defp schedule_work() do
  Process.send_after(self(), :work, 2 * 60 * 60 * 1000)
end
```

`schedule_work/0` is in charge of managing the delay before two rounds
of work.

This function is called in `init/1` to create the first scheduling
when the `GenServer` is started. Then every time `handle_info/2` is
triggered there's a scheduling happening and so on.

To handle the scheduling , `schedule_work/0` use a very simple trick.
It makes use of `Process.send_after/4` to send the `:work` message to
`self()` (our current module) after two hours (time is expressed in
milliseconds).

So every two hours our `handle_info/2` function is called, it does its
processing by calling another module function then it asks
`schedule_work/0` to reschedule the task two hours later.

Really simple and handy right?

# Avoid scheduling drifting #

There's one gotcha in the previous example.

If our `MyApp.SomeModule.do_some_long_job/0` function isn't
asynchronous and takes some time to complete then our next scheduling
will be delayed. There will be a drift in the schedule.

If you really need your scheduling to be accurate, you should move the
`schedule_work/0` call at the beginning of the `handle_info/2`
function.

```elixir
def handle_info(:work, state) do
  schedule_work()
  MyApp.SomeModule.do_some_long_job()
    
  {:noreply, state}
end
```

This way the scheduling will be the first instruction called and it
will be set right away. Then the real work will happen. Two hours
later another computing will be done either the previous one is over
or not.

# Start the scheduler using the supervision tree #

All we have left to do is to add our module to the supervision tree so
it will be started along with the application. In a typical mix
application it would be in `lib/app_name/application.ex`:

```elixir
def start(_type, _args) do
  # List all child processes to be supervised
  children = [
    MyApp.Scheduler
  ]

  # See https://hexdocs.pm/elixir/Supervisor.html
  # for other strategies and supported options
  opts = [strategy: :one_for_one, name: MyApp.Supervisor]
  Supervisor.start_link(children, opts)
end
```

The use-case we just run through is described in [the GenServer
    doc](https://hexdocs.pm/elixir/GenServer.html#module-receiving-regular-messages).
    
Elixir documentation is full of interesting stuff, you should read it
carefully before jumping straight away on an external package to
achieve your goal.
    
Using a `GenServer` and a simple combination of
[handle_info/2](https://hexdocs.pm/elixir/GenServer.html#c:handle_info/2)
and
[Process.send_after/4](https://hexdocs.pm/elixir/Process.html#send_after/4)
is enough for a simple use-case like this one.
    
