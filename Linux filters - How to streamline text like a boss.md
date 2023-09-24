---
title: Linux filters - How to streamline text like a boss
description: What is the linux philosophy and how to follow it by creating small tools that manipulate text and can be put in a pipeline, this is what we'll go through on this artcile.
tags: ruby,shell,beginners,linux,vim
cover_image: ""
canonical_url: 
published: false
---
Are you familiar with the Unix philosophy and how to create better scripts? In this comprehensive guide, we'll explore the general definition of Unix philosophy, investigate the key elements of a well written script, and learn the building blocks of scripting such as the pipeline operator, stdin, and stdout manipulation. Finally, we'll dissect how to apply those as good practices on our ruby/bash scripts!

## Table of Contents
- [What is the Unix philosophy?](#what-is-the-unix-philosophy)
- [What is a pipeline?](#what-is-a-pipeline)
- [What is stdin and stdout?](#what-is-stdin-and-stdout)
- [What defines a bad script and how to turn into a good one?](#what-defines-a-bad-script-and-how-to-turn-into-a-good-one)
	- [Ignoring standard input and output communication](#ignoring-standard-input-and-output-communication)
	- [Doing a lot on the same tool, aka monolithic scripts](#doing-a-lot-on-the-same-tool-aka-monolithic-scripts)
- [Bonus point: The bang operator in vim](#bonus-point-the-bang-operator-in-vim)
- [Conclusion](#conclusion)


## What is the Unix philosophy?
The Unix philosophy was originally defined by the master [Ken thompson](https://en.wikipedia.org/wiki/Ken_Thompson) and it's a set of good practices that define a *minimalist* and *modular* software, all the core utils from Unix (such as `find`, `grep`) follow this good practices so we can agree that it need to be good right?

The original quote documented by [Doug Mcllroy](https://en.wikipedia.org/wiki/Douglas_McIlroy) contain the following items:

> 1. Make each program do one thing well. To do a new job, build afresh rather than complicate old programs by adding new "features".
> 2. Expect the output of every program to become the input to another, as yet unknown, program. Don't clutter output with extraneous information. Avoid stringently columnar or binary input formats. Don't insist on interactive input.
> 3. Design and build software, even operating systems, to be tried early, ideally within weeks. Don't hesitate to throw away the clumsy parts and rebuild them.
> 4. Use tools in preference to unskilled help to lighten a programming task, even if you have to detour to build the tools and expect to throw some of them out after you've finished using them.

The last two items define a more "programming in general" tips, the goal of this article will focus entirely on the first two tips where a good script can be known as *something that do one job really well and can be put in a pipeline*. On the following chapters we'll try to understand this phrase a little better.

## What is a pipeline?

A pipeline is a continuous run of programs where the stdout of one follow as the stdin of the next one, on bash/zsh/fish shells we can use the `|` operator to refer as a pipe and we use it a lot to further query information, for example: imagine you want to count how many markdown files you have in your directory, you can do the following:

```sh
find . -iname '*.md' | wc -l
```

Let's break it down piece by piece so you can understand properly shall we?

First we have the `find` command being used to list all the markdown files on the current directory, for me personally this produces the following:

```sh
$ find . -iname '*.md'
./posts/20230814T124722/README.md
./posts/20230703T214043/README.md
./posts/20230616T234323/README.md
./posts/20230625T223158/README.md
./posts/20230731T212528/README.md
./posts/20230807T203924/README.md
./posts/20230804T140043/README.md
./REPL Driven Development - For not so smart developers.md
./Como escrever uma CLI CRUD utilizando ScyllaDB + Ruby.md
./Linux filters - How to streamline text like a boss.md
```

If you just want to list the proper file names without counting them, you can even pipe it to the `sort` command, that way it lists alphabetically like the following:

```sh
$ find . -iname '*.md' | sort
./Como escrever uma CLI CRUD utilizando ScyllaDB + Ruby.md
./Linux filters - How to streamline text like a boss.md
./REPL Driven Development - For not so smart developers.md
./posts/20230616T234323/README.md
./posts/20230625T223158/README.md
./posts/20230703T214043/README.md
./posts/20230731T212528/README.md
./posts/20230804T140043/README.md
./posts/20230807T203924/README.md
./posts/20230814T124722/README.md
```

See how the output of the first `find` command was streamlined to the `sort` command where it returned it properly sorted? This is the entire purpose of writing small tools, that way you can easily compose them into different pipelines to get cool results.

In the case of the initial example we used the `wc` command that only count whatever it receives from stdin, in this case we use the `-l` flag to count lines. With all that put together voilá! We have our result:

```sh
$ find . -iname '*.md' | wc -l
      10
```

## What is stdin and stdout?

Stdin (standard input) and stdout (standard output) are the main ways that a computer communicates with the outside world. Below, we'll delve more into the details of each one:

- Stdin (standard input) is normally referred to as anything that waits for a user interaction (like typing on an input, selecting an item from a list, etc.), but this can be understood in a more generalized way; basically, we can think of Stdin as the default *information provider*, whether this comes from another program or from a user interacting with it.
- Stdout (standard output) is normally referred to as anything that prints out a value to the screen (it can be a terminal, a browser, or anything). Differently from Stdin, this is totally correct, the only caveat is that when inserted in a pipeline context, all stdout is suppressed as the stdin of the next command instead of printing to the screen.

## What defines a bad script and how to turn into a good one?

Describing a exact summary of what is a good script can be quite tricky because of the many variables involved. Instead, let's flip the question: what does a bad script look like according to the Linux philosophy? And how can we solve the specific problem to create slightly better tools?

### Ignoring standard input and output communication

Have you ever used some CLIs that have a fancy way to display information with an animated input or a cool spinner? It's indeed quite cool to use those tools when you just want to use them by themselves, but when you try to put those tools in a pipeline, they break the whole process. Instead, always write your tools by using standard communication as the primary way to receive and retrieve information, prefer to use options instead of hard-coding values (an example of an option is `--output=json` or `-o json`).

In Ruby, it's far easier to not shoot yourself in the foot because the way we learn to get data already works pretty well with the standard input, but it's important to keep an eye on how you design your script. 

For example, consider this simple script called `printname.rb` example:

```ruby
#!/usr/bin/env ruby

puts 'Type your name: '
name = gets.chomp

puts "Your name is: #{name}"
```

This script in a first glance works both running by itself and on a pipeline, but observe the output produced by both:

```sh
$ echo "Cherry Ramatis" | ./printname.rb
Type your name:
Your name is: Cherry Ramatis
```

```
$ ./printname.rb
Type your name:
Cherry Ramatis
Your name is: Cherry Ramatis
```

You can observe that we're always printing the `Type your name` message, in this case we have two options:

- Accept this name with a flag parameters instead by receiving `--name="Cherry Ramatis"`
- Drop the `Type your name` message, turning the script into a more pipelineable (nice word huh?) one.

### Doing a lot on the same tool, aka monolithic scripts

Sometimes you're writing a tool and quickly observe that the scope got bigger while you were writing it, now what should be a small tool is growing to be a big project with various interactive steps. When it reaches that point it's quite seducing to just keep iterating on the same tool by adding a lot of sub commands, but the unix philosophy help us understand that writing a tool that *do one thing right* is more valuable than writing a gigantic behemoth.

So instead of creating a big CLI with a whole set of commands, try to think in writing small separate tools that interconnect and complement each other, for example:

Imagine you want to write a command that list all the songs from a database in a particular order and prompt the user to choose one, instead of writing all those functionalities in *one* command, you can compose with different ones like:

```sh
$ list_songs | sort | update_song
```

See how `list_songs` and `update_song` are different scripts? both communicate via STDIN and STDOUT, allowing the user to pipeline through any command and receive the same behavior (for example we're piping it to sort because we want to view the list in a sorted manner.)

## Bonus point: The bang operator in vim

Rejoice, Vim users! This is our time to shine. Vim is a somewhat standard editor in Unix systems and this comes with a set of benefits such as interacting with binaries directly in your buffer via the `!` operator.

In vim you can press `!!` in **normal mode** to populate a command like this in your minibuffer `:.!`, the command you type after the `!` will be used with the current line as STDIN, mind blowing right? Observe the example below:

Consider the following script in ruby:

```ruby
#!/usr/bin/env ruby

STDIN.each do |line|
 puts "- #{line}"
end
```

In this script we're looping over the STDIN received and printing it back to the STDOUT with a `-` on the beginning, now we can interact with vim:

<a href="https://asciinema.org/a/607360" target="_blank"><img src="https://asciinema.org/a/607360.svg" /></a>

Other variations of the bang command:
- `!}`: Populate the bang command from the current line to the end of the paragraph.
- `!/pattern`: Populate the bang command from the current line to the first find of `pattern` in the buffer.
- `!G`: Populate the bang command from the current line to the end of the file.
- `!4j`: Populate the bang command from the current line to 4 lines below.

## Conclusion

This is a smaller article where I tried to bring more context into something that I do a lot in my day to day life: **automating**. I hope this content is useful for you and if I can help with anything just reach me out! May the force be with you 🍒
