---
title: Gen servers
description: We as developer love abstractions, what about introducing a cool abstraction on top of the elixir core concept that is running everything in a process? Let's learn more about gen server.
tags:
  - elixir
  - beginners
  - programming
cover_image: ""
canonical_url: 
published: false
---
We already know that everything in elixir is a process and that we can maintain a PID through a long-running process to receive and create states, both by writing our own module or using predefined "data structures" like `Agent` right? Another known fact is that we as developers love to write abstractions over known constructs. So allow me to introduce this cool abstraction around the long-running process called `GenServer`.

```
TODO
- Linkar primeiro artigo sobre state manage
```

## Table of Contents

- [What is Gen server and what problem it solves](#what-is-gen-server-and-what-problem-it-solves)
- [How to create your own Gen server abstraction](#how-to-create-your-own-gen-server-abstraction)
- [How to use the Gen server](#how-to-use-the-gen-server)
- [Conclusion](#conclusion)

## What is Gen server and what problem it solves
It's really important to understand what problem we're trying to solve before we even consider what implementation we should try right? So the problem here is lack of abstraction, today we can write our own module that handle some state and implement a method that use the `receive` block to create a "mailbox" or we can use simple abstractions such as `Agent` and `Task` to either hold state only or manage asynchronous tasks only.

First, let's rewind a bit and understand what we do to share state in a process (just like we did with more detail in the previous article). `TOOD: aqui falta um gancho pro proximo paragrafo`

The `Agent` wrapper works really well for this goal of holding state and it's pretty easy to use, as you can see below in the example:

```elixir
# Creating an agent
{:ok, agent} = Agent.start_link(fn -> [] end)

# Updating the state inside an agent
Agent.update(agent, fn list -> ["cherry" | list] end)

# Retrieving the state inside an agent
Agent.get(agent, fn list -> list end)
Agent.get(agent, fn list -> Enum.find(fn item -> item == "cherry" end) end)
```

Although this is really cool, we're bound to use only for keeping an state inside a process specifically and we miss the opportunity to have a mailbox. But you might be wondering "what is a mailbox anyway?".

`TODO: confirma isso aqui`
A mailbox is a common term in the elixir community that define the usage of `receive` block (under the hood or explicitly) to handle messages from outside the process and execute methods on top of it spawning more processes for each action, it's essentially a way to perform action instead of just holding a data structure state.

Another abstraction that we can use with processes is the `Task` module, it works around the idea of handling asynchronous processes (specifically abstracts the `spawn` function) it has a **really** simple API and it works like the following example:

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

## How to create your own Gen server abstraction
## How to use the Gen server
## Conclusion