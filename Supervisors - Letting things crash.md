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

The supervisor will be responsible to restart the process once it crashes accordingly to the defined strategy (which we'll see more about below), this create a very specific and cool philosophy that exists in the elixir community called "Let things crash"! This basically defines that we should not worry of defining infinite structures to avoid crashing our process (a.k.a try catching all the dway), we should instead just let the process die and be restarted by our supervision tree, this is immensely useful for a functional language like elixir because everything is immutable so we can finish and restart processes at will with no cost at all, cool right?

> TODO: Talvez escrever mais? ou colocar imagens acho, fica mais visual

## Declaring a supervisor

## Creating a mix project with a supervisor already implemented

## Visualizing processes in a graphical way with builtin erlang function

## Conclusion
