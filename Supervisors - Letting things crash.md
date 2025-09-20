---
title: Supervisors - Letting things crash
description: It's time to explore the last bit about managing processes with elixir! We'll understand what is a supervisor, what is a supervision tree and how do we manage our processes once it dies! Let's crash processes together
tags:
  - elixir
  - beginners
  - programming
  - article
published: true
---

In the previous articles, we explored various aspects of managing processes in Elixir. From creating processes for asynchronous tasks to persisting state across long-running processes, and even implementing a message-based architecture for efficient process communication, how much more things we can do with processes in elixir for god’s sake!? Ahem - in this article, we'll delve into another critical aspect of process management in Elixir: handling the lifecycle of processes, especially when they die/are terminated.

Supervisors is a neat architecture/abstraction around processes (as always) that lives above another processes to check the status of our processes and revive any of those based on a custom strategy. With this we can loudly say the classic elixir phrase "Let it crash".

## Table of contents

- [What is a supervisor](#what-is-a-supervisor)
- [Declaring a supervisor](#declaring-a-supervisor)
- [What is a supervision tree and how to visualize it graphically](#what-is-a-supervision-tree-and-how-to-visualize-it-graphically)
- [Creating a mix project with a supervisor already implemented](#creating-a-mix-project-with-a-supervisor-already-implemented)
- [Conclusion](#conclusion)

## What is a supervisor

A supervisor is a process, can you imagine? Everything is a process!!! But this one serves a specific purpose of controlling/supervising other processes to check if they're still running or if they already crashed.

The usage of those supervisors create what we call a _supervision tree_, and it's what drives a lot of big frameworks such as [Phoenix](https://phoenixframework.org) to provide fault-tolerant control and visualization for the process in your application, this give us much more control and performance while trusting the awesome Erlang VM.

The title of this article is "Letting things crash" for one particular reason, this is a very common way of thinking around the Elixir community mostly because of Supervisors, by letting your functions be immutable and run as separate processes, it becomes **really** cheap to just let the process die and be restarted by a supervisor instead of creating a lot more structure around to not simply crash. The language structure works around that idea generally.

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

Since the supervisor is just another process, we should start by defining its own `start_link` function, and remember what we saw above about supervisor accepting any process that implements the `start_link` function? So it's exactly what you're thinking right now, we can pass supervisors to be supervised by other supervisors since it's just processes dealing with processes! Cool right?

The `init` function serves the purpose of instantiating the child processes and define a specific strategy, this strategy defines which way the supervisor will restart the process once it's dead, and we'll see more in detail below:

- `one_for_one`: This is the "standard" and most simple of all the strategies, it simply restarts just the specific child that terminated while supervising the other processes, allowing it to run without change.
- `one_for_all`: This one is a little bit trickier, not only a great power of boku no hero but also a very powerful strategy for keeping multiple processes in sync! Once one process is terminated, all the other remaining are restart together with the original one, this make it **really** easy to keep all the processes in sync if you need to.
- `rest_for_one`: Last but not least, this one is similar to the `one_for_one` with some caveats specific to it. It'll restart the specific process once it's terminated but will also restart all the process that started **after** the failed process (most probably processes started by the failed one), this is useful if you have a centralized process that dispatch actions and need to be alive at the same time as it's dispatched processes.

Once we define this supervisor module, we gain some API to manage the processes under the tree:

- `which_children/1`: This function list all the children under the main supervisor with the respective PIDs, so we can reutilize and manipulate further.
- `start_child/2`: This function starts a particular child programmatically or return a tuple with `{:error, {:already_started, pid}}` if the process is already alive.
- `stop/1`: This function stops the supervisor itself (and all the children under it's tree).

## What is a supervision tree and how to visualize it graphically

![A supervision tree](https://github.com/cherryramatisdev/public_zet/assets/86631177/84630a91-256a-4500-a660-dcadf416889a)

The supervision tree created by using supervisors is what make elixir stand out so much compared to other languages, it's very flexible since a supervisor can take another supervisor as children, but this also make it grow the number of processes running at the same time exponentially.

For dealing with this specific situation, we can use an awesome tool provided natively by the Erlang VM to visualize processes and supervision trees:

The tool is run by calling `:observer.start()` on the REPL, but if you're using as `iex -S mix` this tool don't come for free, and we need to activate it as follows:

```elixir
$ iex -S mix
iex(1)> Mix.ensure_application!(:wx); Mix.ensure_application!(:runtime_tools);Mix.ensure_application!(:observer)
:ok
iex(2)> :observer.start()
:ok
```

A GUI will open where you can view and debug all the details about the running Erlang VM, view the supervision trees, check the memory usage, etc. It's really powerful to rely on this tool while developing big application to make sure you're respecting the best pattern and design decisions!

![Supervisor GUI](https://github.com/cherryramatisdev/public_zet/assets/86631177/5a68ec4c-1410-4bf2-a95c-56f5b2c99f09)

## Creating a mix project with a supervisor already implemented

Since this architecture is pretty common in the elixir world, the known tool `mix` can be used to generate a sample project with a supervision tree already setup, so you can just plug in new processes modules, below we'll understand the more practical pieces:

You can simply run the following command to start a new project:

```sh
$ mix new project_name --sup
```

This will create the following file structure:

```
.
├── README.md
├── lib
│   ├── project_name
│   │   └── application.ex
│   └── project_name.ex
├── mix.exs
└── test
    ├── project_name_test.exs
    └── test_helper.exs

4 directories, 6 files
```

The `application.ex` file define our root supervisor where we can register different processes.

Another thing that this project creation defines is modifying the `mix.exs` to auto start the supervision tree:

```elixir
# Run "mix help compile.app" to learn about applications.
def application do
  [
    extra_applications: [:logger],
    mod: {Teste.Application, []}
  ]
end
```

## Conclusion

And the end goal was reached! This is the last article of the series about process management using elixir and I hope any content here is useful as it was for me. Both the elixir core concepts and the community is amazing, this was truly an awesome experience to have and share with y'all. If I can help with anything, just reach it out and may the force be with you 🍒
