---
title: GenServer with supervision tree and state recovery after crash
category: Elixir
tags: ['elixir']
header:
    overlay_image: /assets/images/headers/elixir.png
excerpt: "You've probably heard that in Elixir you should let processes
    crash rather than trying to catch every possible exceptions. Then,
    after a crash, you only have to get back to the latest
    good known state in the process the supervisor created for you. Great
    but there are few explainations about how to recover the state
    available out there. Let's fix this!"
---

## Why state recovery is important and why do I write about this ##

When I first tried to understand and embrace the principle of "let it
crash" I quickly wondered what was the right way to recover the state
of a
[GenServer](https://elixir-lang.org/getting-started/mix-otp/genserver.html)
after a crash. The most up-to-date known good state rather that
restarting the GenServer from the initial state.

In this post we'll see how to create a GenServer, monitor it using a
supervision tree then how to handle its state so when it crashes it
can be restarted using the latest known state.

## Sequence server without state recovery ##

### What are we going to do? ###

To illustrate this topic, we're going to write a sequence server.

Its purpose is to respond to the caller with a number, increment it
and wait for another call.

We'll also allow the user to manually increment the sequence by a
delta of its choice.

### Create a fresh Mix project ###

Let's create a new project using [Mix](https://elixir-lang.org/getting-started/mix-otp/genserver.html):

``` shell
$ mix new --sup sequence
* creating README.md
* creating .formatter.exs
* creating .gitignore
* creating mix.exs
* creating lib
* creating lib/sequence.ex
* creating lib/sequence/application.ex
* creating test
* creating test/test_helper.exs
* creating test/sequence_test.exs

Your Mix project was created successfully.
You can use "mix" to compile it, test it, and more:

    cd sequence
    mix test

Run "mix help" for more commands.
```

Now we have a full blown Elixir application with formatter config,
tasks, test files and a `lib` directory to host our upcoming code, we
can start to write our mind-blowing server!

### Application is the root of everything ###

When we'll start our application, our entry point will be the
[Application](https://hexdocs.pm/elixir/Application.html) module.

Every single Elixir app works this way. There's a module that `use
Application` which defines a standardized directory structure,
configuration and lifecycle.

Our `Application` module is located at `lib/sequence/application.ex`,
let's take a look at it:

``` elixir
defmodule Sequence.Application do
  # See https://hexdocs.pm/elixir/Application.html
  # for more information on OTP Applications
  @moduledoc false

  use Application

  def start(_type, _args) do
    children = [
      # Starts a worker by calling: Sequence.Worker.start_link(arg)
      # {Sequence.Worker, arg}
    ]

    # See https://hexdocs.pm/elixir/Supervisor.html
    # for other strategies and supported options
    opts = [strategy: :one_for_one, name: Sequence.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

As you can see in the `start` function, a supervisor will watch for all
our `children` but we don't have any for now.

### The naive sequence server ###

Time to write some code!

The `Sequence.Server` module will store and increment a value, we need
create it and then add it to the supervision tree.

Open up the `lib/sequence/server.ex` file to write our server
implementation:

``` elixir
defmodule Sequence.Server do
  use GenServer

  # public API

  def start_link(current_number) do
    {:ok, _pid} = GenServer.start_link(__MODULE__, current_number, name: __MODULE__)
  end

  def next_number do
    GenServer.call(__MODULE__, :next_number)
  end

  def increment_number(delta) do
    GenServer.cast(__MODULE__, {:increment_number, delta})
  end

  # GenServer callbacks

  def init(initial_state) do
    {:ok, initial_state}
  end
  
  def handle_call(:next_number, _from, current_number) do
    { :reply, current_number, current_number + 1 }
  end

  def handle_cast({:increment_number, delta}, current_number) do
    { :noreply, current_number + delta }
  end

  def format_status(_reason, [ _pdict, state ]) do
    [data: [{'State', "Current state: '#{inspect state}"}]] 
  end
end
```

The goal of this blog post is not to explain how GenServer or Elixir
works but I want to make sure everyone understand what happens in this
module.

First our module take advantage of `use GenServer` so it can easily
talk with external processes, receive synchronous and asynchronous
calls, etc.

Next you can see I commented that I structured the code into two main
parts, the public interface and a more private one which encapsulate
the details of working with a GenServer and its callbacks.

Our public API expose the needed tooling for our client to initialize
a `Sequence.Server`, the `start_link` function, there also a
`next_number` and `increment_number` that allow to do the real job. In
fact those two functions will only delegate to GenServer `call` and
`cast` function that will use our private API.

The private API implements the `init` function that will be call by
`GenServer.start_link` to set the initial set of our server.

There's a `handle_call` function definition that pattern match on the
`:next_number` argument. This function receive the caller info that we
won't use and the current state of the server, that is the current
number.

All this function does is to reply with the `current_number`, and
store `current_number + 1` as the new state.

The last function `handle_cast` pattern match on `{:increment_number,
delta}` tuple, its second argument is the current server state. This
function doesn't send a response to the caller. Its job is to
increment `current_number` of the `delta` value and store it as the
new state.

Oh and there's another function called `format_status`, it's a
standard function of GenServer that is call when the server crashes
for instance. It will be useful for testing purpose.

### Adding our server to the supervision tree ###

Now we have a module that does the sequencing job, we can add it to
the supervision tree:

``` elixir
def start(_type, _args) do
  children = [
  # Starts a worker by calling: Sequence.Worker.start_link(arg)
  {Sequence.Server, 0}
  ]

  # See https://hexdocs.pm/elixir/Supervisor.html
  # for other strategies and supported options
  opts = [strategy: :one_for_one, name: Sequence.Supervisor]
  Supervisor.start_link(children, opts)
example
```

So now when our application starts, it will create a process that runs
our `Sequence.Server` with the default state equals to zero. We used
the strategy `one_for_one` so if our process crashed, the supervisor
is going to start a fresh new one.

### Trying out our server ###

To test-drive our new sequence module we are going to use the Elixir
REPL <kbd>iex</kbd>:

``` shell
$ iex -S mix
iex(1)> Sequence.Server.increment_number(12)
:ok
iex(2)> Sequence.Server.next_number
12
iex(3)> Sequence.Server.increment_number(3)
:ok
iex(4)> Sequence.Server.next_number
16
iex(5)> Sequence.Server.next_number
17
```

Our sequence server is working great! But it's not very robust.

If we call `increment_number` with something that is not a number,
it'll crash. But since we have a supervisor and a strategy defined it
should restart:

``` shell
iex(6)> Sequence.Server.increment_number(nil)
:ok
iex(7)> 
00:40:56.745 [error] GenServer Sequence.Server terminating
** (ArithmeticError) bad argument in arithmetic expression
    :erlang.+(8, nil)
    (sequence) lib/sequence/server.ex:28: Sequence.Server.handle_cast/2
    (stdlib) gen_server.erl:637: :gen_server.try_dispatch/4
    (stdlib) gen_server.erl:711: :gen_server.handle_msg/6
    (stdlib) proc_lib.erl:249: :proc_lib.init_p_do_apply/3
Last message: {:"$gen_cast", {:increment_number, nil}}
State: [data: [{'State', "Current state: '8'"}]]
```

It crashed as expected. Now what happens if we try to use it again?

``` shell
iex(8)> Sequence.Server.next_number
0
```

Damn! Our sequence process has restarted but… the state was lost. We
started over at 0, the initial value of the sequence server.

We need to find a way to keep the latest value across sequence server
crashes. After all that's the whole point of this blog post!

## Keeping the state safe ##

If our GenServer was stateless we'd be good and we can keep writing more
features. But in our example preserving state if the sequence server
crashes is a main concern.

So how can we keep our state across GenServer crashes to be able to
send an up-to-date value?

It's ok that your inner code crashes and it won't have a real bad
effect on your client. What matters is that you're able to provide
your clients with data as fresh as possible.

Our supervisor simply restart our sequence server with its initial
state. Preserving processes state is not its jobs.

So what's the best way to be able to use an up-to-date state in our
sequence server?

Don't think too much, choose the easy path. You should use a separate
process to handle state. With separate processes you don't care if the
sequence server crashes since the state is stored somewhere else.

Now let's write a module which only purpose will be to store the
state.

```elixir
defmodule​ Sequence.Stash ​do​​
  use​ Agent

  def​ start_link(initial_value) ​do​
    Agent.start_link(fn -> initial_value end, name: __MODULE__)​
q  end​

  def​ get() ​do​
    Agent.get(__MODULE__, &(&1))​
  end​

  def​ update(new_value) ​do​
    Agent.update(__MODULE__, new_value)
  end​
end​
```

[Agents](https://elixir-lang.org/getting-started/mix-otp/agent.html)
are an abstraction in Elixir dedicated to store and share state across
processes. That's exactly what we need!

Since we use the good tool for the good job, the code we had to write
is really short and simple.

We had to start the agent with `start_link`, implement a way to
retrieve the current value to do so we used `Agent.get` providing the
current module name as first argument and an anonymous function which
simply returns the value of the first argument it is provided with
that is the current state of the agent, our counter.

We also implemented a way to update the agent state by using
`Agent.update`, still with the current module name as first argument
and the new value to store as second argument.

### Supervising stash server ###

Now we have a dedicated module to store our sequence value, we need to
start it with the application and ensure it will always run.

We can be pretty confident with this module robustness. Its code is so
simple that it would be hard to crash it.

So let's add it to the supervision tree.

As you may remember, we used `one_for_one` strategy for our sequence
server but there are other strategies available.

When you add a module to the supervision tree, you should always ask
yourself what is the strategy that better fits your needs.

There are three main strategies:

- `one_for_one` restarts the process and only this one if it crashes
- `one_for_all` restarts all the child processes when this one crashes
- `rest_for_one` restarts the crashed process and all those that were
  started after him
  
Depending of your needs and how you built your dependencies between
your modules, you should choose the right one. There's also
`DynamicSupervisor` that is suited to attached children to the
supervision tree dynamically but that's beyond the scope of this blog
post.

If your module can be killed and restarted without impacting anything
else you should go for `one_for_one`.

If your module crashed and that it would create a inconsistency across
the app after it restarted, you should go for `one_for_all`.

If only modules declared after the one that crashed would be affected
you should go for `rest_for_one`. 

Enough talking, get back to the code and our supervisor:

``` elixir
defmodule Sequence.Application do
  @moduledoc false
  
  use Application
  
  def start(_type, _args) do
    children = [
      { Sequence.Stash, 0 },
      { Sequence.Server, nil }
    ]
    
    opts = [strategy: :rest_for_one, name: Sequence.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

So we added `Sequence.Stash` to the supervision tree, at the first
position, with a default value of `0`. We added it first because our
`Sequence.Server` relies on it. It makes sense in my opinion.

We changed the default state of `Sequence.Server` to `nil` since this
module doesn't need to handle its state for itself. That's the
`Sequence.Stash` job.

If you looked closely, you'll noticed we have changed our strategy
from `one_for_one` to `rest_for_one`. This is not a needed change. Our
code would have worked without it. This is more of a way to express
the intention. It's here to say "hey if Stash crashes we loose
everything and have to restart".

## Updating sequence server to use stash server ##

Now we need to update our `Sequence.Server` to make use of our new
module that handles the state.

It's going to be easy:

### code example ###

``` elixir
defmodule Sequence.Server do
  use GenServer

  # Public API

  def start_link(stash_pid) do
    {:ok, _pid} = GenServer.start_link(__MODULE__, stash_pid, name: __MODULE__)
  end
  
  def next_number do
    GenServer.call(__MODULE__, :next_number)
  end
  
  def increment_number(delta) do
    GenServer.cast(__MODULE__, {:increment_number, delta})
  end

  # GenServer implementation

  def init(stash_pid) do
    current_number = Sequence.Stash.get_value(stash_pid)
    { :ok, {current_number, stash_pid} }
  end
  
  def handle_call(:next_number, _from, {current_number, stash_pid}) do 
    { :reply, current_number, {current_number + 1, stash_pid} }
  end
  
  def handle_cast({:increment_number, delta}, {current_number, stash_pid}) do
    { :noreply, {current_number + delta, stash_pid}}
  end
  
  def terminate(_reason, {current_number, stash_pid}) do
    Sequence.Stash.save_value(stash_pid, current_number)
  end
end
```

As you can see our module now makes use of `Sequence.Stash` to store
value. By doing this we can delegate the value storing to another
module. This module logic is very simple and thus it avoid unexpected
crash. It's the best way to ensure your Elixir code robustness.

Now let's see what happens when it crashes.

### Seeing crash and self-healing in action ###

``` elixir
$ iex -S mix
​iex>​ Sequence.Server.next_number
0
​iex>​ Sequence.Server.next_number
1
​iex>​ Sequence.Server.next_number
2
​iex>​ Sequence.Server.increment_number ​"​​cat"​
:ok
​iex>​
12:15:48.424 [error] GenServer Sequence.Server terminating
​**​ (ArithmeticError) bad argument in arithmetic expression
​iex>​ Sequence.Server.next_number
3
​iex>​ Sequence.Server.next_number
4
​iex>​ Sequence.Server.next_number
5​
:ok​
```

## Final words ##

I think that this OTP feature is a really cool way to handle
unexpected crashes. 

You now have code that can crash and recover without losing its
previous state. 

We use an Agent here to store the value in memory but the exact same
technique could be used to store and retrieve value using a database, a
file on the disk, a web service, …
