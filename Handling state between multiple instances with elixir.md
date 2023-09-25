---
title: Handling state between multiple instances with elixir
description: Elixir code works really well for concurrent code, want to know why? Let's dive in the world of processes in elixir with tasks, agents and gen servers.
tags: beginners,elixir,tutorial
cover_image: ""
canonical_url: 
published: false
---

Elixir works **really** well for concurrent code because of it's functional nature and ability to run in multiple processes, but how we handle state when our code is running all over the place? Well, there is some techniques and in this article we'll learn more about it together shall we?

## Table of contents
- [What is a process? How to use it with send and receive](#what-is-a-process-how-to-use-it-with-send-and-receive)
- [Incrementing our experience with tasks](#incrementing-our-experience-with-tasks)
- [Designing state with the agent wrapper](#designing-state-with-the-agent-wrapper)
- [Conclusion](#conclusion)

## What is a process? How to use it with send and receive

![image](https://github.com/cherryramatisdev/public_zet/assets/86631177/725725cb-003e-4659-943a-c7978e74281c)

Processes are the answer from Elixir to concurrent programming; they're basically a continuous-running node that can send and receive messages. In fact, every function in Elixir runs inside a process. Although this sounds really expensive, it's **super** lightweight compared to threads in other languages, which empowers us developers to build incredibly scalable software with hundreds of processes running at the same time. Another great advantage of using this specifically with the Elixir language is that this language is built on top of immutability and other functional programming concepts, so we can trust that these functions are running completely isolated and without changing or maintaining global state.

The basic way of seeing a process in action is by using the `spawn` function, with that we can execute a function in a process and get the *pid* of it.

```elixir
iex(3)> pid = spawn(fn -> IO.puts("teste") end)
teste
#PID<0.111.0>
iex(4)> pid
#PID<0.111.0>
iex(5)> Process.alive?(pid)
false
iex(6)>
```

As you can see from the return of `Process.alive?(pid)` this process is already dead once it run correctly, but we can easily add a sleep function to check this mechanism:

```elixir
iex(2)> pid = spawn(fn -> :timer.sleep(10000); IO.puts("teste") end)
#PID<0.111.0>
iex(3)> Process.alive?(pid)
true
teste
iex(4)> Process.alive?(pid)
false
iex(5)>
```

Since we're sleeping for 10 seconds, the process is alive until it run after the sleep and dies. Cool right? It's important to know that *our main program did not hang*, it simply put the function in a process and forgot there, this allow us to create really modular and performant code abusing to run on multiple nodes.

Besides spawning functions in a process, we can transition information between processes using the functions `send` and the `receive` block as shown below:

```elixir
iex(1)> defmodule Listener do
...(1)> def call do
...(1)> receive do
...(1)> {:hello, msg} -> IO.puts("Received: #{msg}")
...(1)> end
...(1)> end
...(1)> end
{:module, Listener,
 <<70, 79, 82, 49, 0, 0, 6, 116, 66, 69, 65, 77, 65, 116, 85, 56, 0, 0, 0, 240,
   0, 0, 0, 25, 15, 69, 108, 105, 120, 105, 114, 46, 76, 105, 115, 116, 101,
   110, 101, 114, 8, 95, 95, 105, 110, 102, 111, ...>>, {:call, 0}}
iex(2)> pid = spawn(&Listener.call/0)
#PID<0.115.0>
iex(3)> send(pid, {:hello, "Hello World"})
Received: Hello World
{:hello, "Hello World"}
iex(4)>
```

Observe that we define a function that act as a general listener using the `receive` block, this work as a switch case where we can pattern match and do a quick action, in this case we're simply printing to the STDOUT. Once we spawn this listener, it's possible to use the returned `pid` to send information using the `send/2` function that expects a `PID` and a value as arguments.

That way, it's possible to keep state in an immutable and separate environment such as elixir.

## Incrementing our experience with tasks

This module offers an abstraction on top of the `spawn` function while adding support for asynchronous behavior, i.e. creating function in a separate process and observing it's behavior with wait functions. As you delve into Elixir, you'll discover that the `Task` module allows you to start a new process that executes a function and returns a task structure. With this structure in hand, you can easily get the value from this function using the `Task.await(task)` clause, as shown below: 

```elixir
iex(1)> task = Task.async(fn ->
...(1)>   IO.puts("Task is running")
...(1)>   42
...(1)> end)
Task is running
%Task{
  mfa: {:erlang, :apply, 2},
  owner: #PID<0.109.0>,
  pid: #PID<0.110.0>,
  ref: #Reference<0.0.13955.659691257.723058689.43945>
}
iex(2)> IO.puts "a code"
a code
:ok
iex(3)> answer_to_everything = Task.await(task)
42
iex(4)> answer_to_everything
42
iex(5)>
```

First we saw the `Task is running` message printed out and then we got the task struct, further we could execute any code in between and when we're ready is just a matter of using the `Task.await` function to retrieve the function return.

Task also provide a common interface for the regular `spawn` function called `start`, we can even reutilize the code shown on the beginning with the new module abstraction:

```elixir
iex(1)> defmodule Listener do
...(1)> def call do
...(1)> receive do
...(1)> {:print, msg} -> IO.puts("Received message: #{msg}")
...(1)> end
...(1)> end
...(1)> end
{:module, Listener,
 <<70, 79, 82, 49, 0, 0, 6, 244, 66, 69, 65, 77, 65, 116, 85, 56, 0, 0, 0, 245,
   0, 0, 0, 26, 15, 69, 108, 105, 120, 105, 114, 46, 76, 105, 115, 116, 101,
   110, 101, 114, 8, 95, 95, 105, 110, 102, 111, ...>>, {:call, 0}}
iex(2)> {:ok, pid} = Task.start(&Listener.call/0)
{:ok, #PID<0.115.0>}
iex(3)> send(pid, {:print, "Eat more fruits"})
Received message: Eat more fruits
{:print, "Eat more fruits"}
```

It's useful to use the `Task` module because we can get a higher level of abstraction, you must have noticed that the interface for `Task.start` and `Task.async` is the same? Yeah we can swap those and get the power of using `Task.await` and `Task.yield` on top of it, *that's the power of abstracting lower level concepts!*

## Designing state with the agent wrapper

![image](https://github.com/cherryramatisdev/public_zet/assets/86631177/306a155b-2cc1-4af7-8a52-7aa053bf5680)

The `Agent` module provide another layer of abstraction focused on controlling state between multiple instances of a process, it act like a data structure for long running interactions.

We can first start a agent instance with a initial value passed from a function return as shown below:

```elixir
iex(1)> {:ok, agent} = Agent.start_link(fn -> [] end)
{:ok, #PID<0.110.0>}
iex(2)> agent
#PID<0.110.0>
iex(3)>
```

As you can see we get a `PID` just like the other abstractions, the difference here can be observed on the usage of other methods

For example we can update the original array by appending a value to it:

```elixir
iex(3)> Agent.update(agent, fn list -> ["elixir" | list] end)
:ok
iex(4)>
```

**That's the whole difference** of the agent abstraction, we can continuously update a state by appending immutable functions as callback and reusing the same `PID`.

We also can return a particular value from the data structure by using the following function:

```elixir
iex(4)> Agent.get(agent, fn list -> list end)
["elixir"]
iex(5)>
```

See? it's as simple as returning the whole list from the callback function, you can imagine that it's possible to use any method from elixir to filter down this list if wanted and keep iterating over the data structure

## Conclusion

This is a simple introduction to this concept that is new for me, I hope it's useful for anyone reading it! And in the next articles we'll dive deeper about other topics in elixir such as Gen Servers, Supervisors, etc...
