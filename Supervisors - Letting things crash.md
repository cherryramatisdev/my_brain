---
title: Supervisors - Letting things crash
description: ..
tags: elixir,beginners,programming
published: false
---

> TODO: Revisar esse paragrafo

In the previous articles we've seen how to create processes for asynchronous tasks, how to persist state through a long running process and even how to implement a message based architecture for communication with a process, how much more things we can do with processes in elixir for god sake!? Uh hu hu _clears throat_, the last action on the great circle of life is dealing with the death of a process and this is precisely what we'll look at this article; Supervisors is a neat architecture that defines strategies for reviving our processes after they die and with that we can loudly say the elixir phrase "Let it crash".

## Table of contents

- [What is a supervisor](#what-is-a-supervisor)
- [Declaring a supervisor](#declaring-a-supervisor)
- [Creating a mix project with a supervisor already implemented](#creating-a-mix-project-with-a-supervisor-already-implemented)
- [Visualizing processes in a graphical way with builtin erlang function](#visualizing-processes-in-a-graphical-way-with-builtin-erlang-function)
- [Conclusion](#conclusion)

## What is a supervisor

A supervisor is a process, can you imagine? Everything is a process!!! But this one serves a specific purpose of controlling/supervising other processes to check if they're still running or if they already crashed, the usage of those supervisors create what we call a _supervision tree_ and it's what drives a lot of big frameworks such as [Phoenix](https://phoenixframework.org) to provide fault tolerant control and visualization for the process in your application (which is a lot since we write elixir right?).

The supervisor will be responsible to restart the process once it crashes accordingly to the defined strategy (which we'll see more about below), this create a very specific and cool philosophy that exists in the elixir community called "Let things crash"! This basically defines that we should not worry of defining infinite structures to avoid crashing our process (a.k.a try catching all the way), we should instead just let the process die and be restarted by our supervision tree, this is immensely useful for a functional language like elixir because everything is immutable so we can finish and restart processes at will with no cost at all, cool right?

> TODO: Talvez escrever mais? ou colocar imagens acho, fica mais visual

## Declaring a supervisor

The supervisor abstraction behavior works by accepting other processes that implement the `start_link` function (gen servers basically) in a list of modules, so it's pretty simple to declare it as we can see below:

```elixir
defmodule OurSupervisor do
  use Supervisor

  def start_link(opts) do
    Supervisor.start_link(__MODULE__, :ok, opts)
  end

  @impl true
  def init(:ok) do
    children = [
      SomeGenServer
    ]

    Supervisor.init(children, strategy: :one_for_one)
  end
end
```

Since the supervisor is just another process we should start by defining it's own `start_link` function, and remember what we saw above about supervisor accepting any process that implements the `start_link` function? So it's exactly what you're thinking right now, we can pass supervisors to be supervised by other supervisors since it's just processes dealing with processes! Cool right?

The `init` function serves de purpose of instantiating the child processes and init with a specific strategy, this strategy defines which way the supervisor will restart the process once it's dead and we'll see more in detail below:

- `one_for_one`: This is the "standard" and most simple of all the strategies, it simply restart just the specific child that terminated while supervising the other processes allowing it to run without change.
- `one_for_all`: This one is a little bit trickier, not only a great power of boku no hero but also a very powerful strategy for keeping multiple processes in sync! Once one process is terminated, all the other remaining are restart together with the original one, this make it **really** easy to keep all the processes in sync if you need to.
- `rest_for_one`: Last but not least, this one is similar to the `one_for_one` with some caveats specific to it. It'll restart the specific process once it's terminated but will also restart all the process that started **after** the failed process (most probably processes started by the failed one), this is useful if you have a centralized process that dispatch actions and need to be alive at the same time as it's dispatched processes.

## Creating a mix project with a supervisor already implemented

## Visualizing processes in a graphical way with builtin erlang function

## Conclusion
