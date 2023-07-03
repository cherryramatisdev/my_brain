---
title: Bonzai and how to create a personal CLI to rule them all
description: How to human friendly CLIs with tab completion by default using bonzai and golang
tags: 'go,golang,cli,bonzai'
cover_image: ''
canonical_url: null
published: false
---

One example of a meticulously designed and highly effective command-line interface (CLI), in my opinion, is `git`. Its commands are remarkably intuitive and expressive, considering the time of its creation. Since delving into terminals and automation, I've often thought about creating my own CLI that functions as a personal assistant, capable of holding all the automation scripts I develop. To my surprise, I stumbled upon an awesome developer named [rob](https://github.com/rwxrob) who has created a amazing tool for Go that allows me to do just that! And with a bonus provided by golang itself that is: "Build one binary and carry everywhere you go"

On this article I'll try to present a brief introduction to this framework, it's advantages and of course we'll develop a little CLI as an example.

## What is bonzai

I think this can be explained in more detail on it's own [documentation](https://github.com/rwxrob/book-bonzai), but to sum up bonzai is a framework for go that allows you to create composable CLI with nested commands and tab completion **by default** on each level of the command, with this tool it's possible to create commands like:

```sh
mycli todo add help
```

This command would print out the help information for the `add` command and the cool part with this is that you can get tab completion for every part.

> DISCLAIMER: the tab completion only works on bash.

## Let's create a quick CLI

The best way to learn something is by doing it so let's create a quick and simple CLI:

### Create a folder for your project and start it with `go mod init <yourpath>`

This is required for any go project and you should be familiar with!

### Install the required dependencies

```sh
go get -u github.com/rwxrob/bonzai
go get -u github.com/rwxrob/bonzai/z
go get -u github.com/rwxrob/help
```

> DISCLAIMER: The help package is itself a bonzai branch! It's purpose is to help us provide documentation messages on each level.

### Create the `main.go` file inside `cmd/yourproject/main.go`:

This is a good practice so you can easily install it with `go install ./cmd/yourproject`.

Your main file should look like this:

```go
package main

func main() {
  yourproject.Cmd.Run()
}
```

Pretty simple right? The real coolness comes next though!

### Define the root command file as `cmd.go`

This file will live on the root of our project and should look like this:

```go
package yourproject

import (
▷┅Z "github.com/rwxrob/bonzai/z"
  "github.com/rwxrob/help"
▷┅"github.com/rwxrob/pomo"
)

var Cmd = &Z.Cmd{
▷┅Name:        "pixie",
▷┅Summary:     "A general purpose command line tool",
▷┅Usage:       "",
▷┅Version:     "0.0.1",
▷┅Description: "A general purpose command line tool",
▷┅Site:        "https://github.com/cherryramatisdev/pixie-cli",
▷┅Source:      "https://github.com/cherryramatisdev/pixie-cli",
▷┅Issues:      "https://github.com/cherryramatisdev/pixie-cli/issues",
▷┅Commands:    []*Z.Cmd{help.Cmd},
}
```

As you can see we're using the `Z` package to define the root command and the `help` package as a branch of bonzai, if you want to create a subcommand you just need to create another instance of `&Z.Cmd{}` and insert on the parent array of `Commands` and it'll be automatically tab completable. Pretty simple and amazing in my opinion.

Running our project with `go run ./cmd/yourproject/main.go` you should have this message:

```
```
