---
title: Gen servers - Abstracting state management and task run together
description: We as developer love abstractions, what about introducing a cool abstraction on top of the elixir core concept that is running everything in a process? Let's learn more about gen server.
tags:
  - elixir
  - beginners
  - programming
  - article
cover_image: ""
canonical_url: 
published: true
---

In Elixir, every function runs inside a separate process, and we can deal with processes in a very natural way offered by the language. It's possible to manage state between multiple long-running processes through known abstractions such as `Agent`. In this article, we'll talk a little about an abstraction that allows the management of long-running processes with state and message communication for running functions accordingly! I hope it'll be fun and easy to understand.

## Table of Contents

- [What is Gen server and what problem it solves](#what-is-gen-server-and-what-problem-it-solves)
- [How to use the Gen server](#how-to-use-the-gen-server)
- [References](#references)
- [Conclusion](#conclusion)

## What is Gen server and what problem it solves

It's really important to understand what problem we're trying to solve before we even consider which implementation we should try, right? So the problem here is lack of abstraction around long running processes. Today, we can write our own module that handles some state and implement a method that uses the `receive` block to create a "mailbox", or we can use simple abstractions such as `Agent` and `Task` to either hold state only or manage asynchronous tasks only.

First, let's rewind a bit and understand what we do to share state in a process using simpler abstractions. If you want to know more in detail about those modules, reach out to the previous article linked at the beginning. :)
### Agent

The `Agent` module provides a simple and efficient way to manage state, personally I view this module as an data structure for the process architecture because you can control any primitive value through a pid and getters/setters. It's pretty easy to use as you can see below in the example:

```elixir
# Creating an agent
{:ok, agent} = Agent.start_link(fn -> [] end)

# Updating the state inside an agent
Agent.update(agent, fn list -> ["cherry" | list] end)

# Retrieving the state inside an agent
Agent.get(agent, fn list -> list end)
Agent.get(agent, fn list -> Enum.find(fn item -> item == "cherry" end) end)
```

As you can see, we have a getter and a setter method that both accept a function callback to manage the internal state, with this simple API we can manage it just an data structure.

### Task

Another abstraction that we can use with processes is the `Task` module that works around the idea of handling asynchronous processes (specifically abstracting the `spawn` function). It has a **really** simple API, and works like the following example:

```elixir
# To initiate a new task
task = Task.async(fn ->
  # Do whatever you want here
  IO.puts("Task is running")
  
  42
end)

# Get the return value of this particular task
answer_to_everything = Task.await(task)
```

If you know languages like JavaScript or csharp, you may recall the `async` `await` syntax from them.

After all this, you may be wondering why we need to use a Gen Server if we already have those modules presented above. Well, using a Gen server is similar to gluing all those concepts together (both async processes and state management) with a single and easy-to-use API and with a very important concept implemented: a `mailbox` for message-based architecture.

> A mailbox is a common term in the elixir community that describes a module that can receive messages and act on it. The common way is by using the `receive` block (under the hood or explicitly) to handle those messages via pattern matching and spawn another processes with the correct action to take.

While using this abstraction, we're able to separate our client and server code by defining `handle` methods for each message and acting correctly on them. We'll see more about it below!

## How to use the Gen server

The Gen Server abstraction provides an architecture that allows us to separate our module code into message senders and handle receivers. It's possible to either use `GenServer.call` or `GenServer.cast` with a message atom to activate a specific method via pattern matching.

For example, while using the `Agent` module, you'll most likely keep all the logic in the same place, as shown below:

```elixir
def put(bucket, key, value) do
  # Here is the client code
  Agent.update(bucket, fn state ->
    # Here is the server code
    Map.put(state, key, value)
  end)
  # Back to the client code
end
```

Since the `Agent` module didn't provide enough abstraction, you're forced to provide the function that needs to take action on the state right there in the `update` function. But by using a message-base abstraction, we can modularize our code a lot more, like below:

```elixir
# Client code
def put(pid, key, value) do
  GenServer.call(pid, {:insert, key, value})
end

# Server code
def handle_call({:insert, key, value}, _from, state) do
  {:reply, :ok, Map.put(state, key, value)}
end
```

See? We were able to separate a lot more code by simply using the power of Elixir pattern matching and the message architecture that the Gen server provided. 

The usage of this abstraction is quite straightforward; you need to implement the [behavior](https://elixir-lang.org/getting-started/typespecs-and-behaviours.html#behaviours) as follows:

```elixir
defmodule Example do
  use GenServer

  # Client world (we'll add later)
  
  # Server world

  @impl true
  def init(state) do
    {:ok, state}
  end

  @impl true
  def handle_call(:list_all, _from, state) do
    {:reply, state, state}
  end

  @impl true
  def handle_cast({:insert, name}, state) do
    :timer.sleep(5000)

    {:noreply, [name | state]}
  end
end
```

- The `init` function will be called when we start the Gen Server with `GenServer.start_link`. This works as our entry point to define the initial state that will be managed.
- The `handle_call` function enters the realm of "mailbox" receivers, this function will be evoked when we use `GenServer.call` with a specific action that will be pattern-matched (`:list_all` in the example). It's also important to point out that these functions will be run **synchronously.**
- The `handle_cast` function serves the same purpose as the `handle_call`; it'll be called from a `GenServer.cast` call, and its pattern is matched correctly. The important difference is that this function will run **asynchronously**.

After understanding the server part, let's complement it with some client-code API for us to call on the REPL:

```elixir
defmodule Example do
  use GenServer

  # Client world

  def add_name(pid, name) do
    GenServer.cast(pid, {:insert, name})
  end

  def list_names(pid) do
    GenServer.call(pid, :list_all)
  end

  # Server world
  # ...
end
```

The client code in this specific case will serve as an easy API for the external world, so we don't need to keep referring to the `GenServer` module always. With this set of methods we can handle sync code (the list_names method with `call`) and async code (the add_name method with `cast`) easily!

Observe also that on our `handle_cast` for `:insert` we're using the timer to sleep 5 seconds on purpose!, below we'll show how this works on the REPL to understand the difference between async and sync calls.

> To view the correct delay, watch the below video:

<a href="https://asciinema.org/a/614453" target="_blank"><img src="https://asciinema.org/a/614453.svg" /></a>

To refer to the methods used on the REPL:

```elixir
iex(1)> {:ok, pid} = GenServer.start_link(Example, [])
{:ok, #PID<0.135.0>}
iex(2)> Example.list_names(pid)
[]
iex(3)> Example.add_name(pid, "Cherry")
:ok
iex(4)> Example.list_names(pid)
["Cherry"]
```

The `add_name` method returned instantly, but the `list_names` method suffered from the timer that we imposed. The balance between those two concepts turn this abstraction really useful and flexible!

## References

- https://elixir-lang.org/getting-started/mix-otp/genserver.html
- https://dev.to/reichert621/learning-elixir-s-genserver-with-a-real-world-example-5fef
- https://hexdocs.pm/elixir/1.12/GenServer.html

## Conclusion

This is another part of the learning through processes in elixir! I hope it's useful for anyone reading this, it's been an awesome experience for me since I'm writing while learning the language and all those new concepts. If I can help with anything, just reach out ! May the force be with you 🍒
