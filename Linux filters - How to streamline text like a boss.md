---
title: Linux filters - How to streamline text like a boss
description: What is the linux philosophy and how to follow it by creating small tools that manipulate text and can be put in a pipeline, this is what we'll go through on this artcile.
tags: 'ruby,shell,beginners,linux'
cover_image: ''
canonical_url: null
published: false
---
Are you familiar with the Unix philosophy and how to create better scripts? In this comprehensive guide, we'll explore the general definition of Unix philosophy, investigate the key elements of a well-written script, and learn the building blocks of scripting such as the pipeline operator, stdin, and stdout manipulation. Finally, we'll dissect how to write a script applying all those concepts shown below, both in bash and ruby, to improve your tool belt of technologies!

## What is the Unix philosophy?
The unix philosophy was originally defined by the master [Ken thompson](https://en.wikipedia.org/wiki/Ken_Thompson) and it's a set of good practices that define a *minimalist* and *modular* software, all the core utils from unix (such as `find`, `grep`) follow this good practices so we can agree that it need to be good right?

The original quote documented by [Doug Mcllroy](https://en.wikipedia.org/wiki/Douglas_McIlroy) contain the following items:

> 1. Make each program do one thing well. To do a new job, build afresh rather than complicate old programs by adding new "features".
> 2. Expect the output of every program to become the input to another, as yet unknown, program. Don't clutter output with extraneous information. Avoid stringently columnar or binary input formats. Don't insist on interactive input.
> 3. Design and build software, even operating systems, to be tried early, ideally within weeks. Don't hesitate to throw away the clumsy parts and rebuild them.
> 4. Use tools in preference to unskilled help to lighten a programming task, even if you have to detour to build the tools and expect to throw some of them out after you've finished using them.

The last two items define a more "programming in general" tips, the goal of this article will focus entirely on the first two tips where a good script can be known as *something that do one job really well and can be put in a pipeline*. On the following chapters we'll try to understand this phrase a little better.

## What makes a good script?
## Learning the building blocks
### What is the pipeline operator?
### What is stdin/stdout and how to manipulate it properly