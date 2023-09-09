---
title: Enhancing development with REPLs - A practical guide
description: Have you ever encountered a problem and immediately had the solution pop into your mind? No? So this article is for you! Let's learn how to use the REPL in our favor for development.
tags: ruby,repl,development
published: false
---

Have you ever encountered a problem and immediately had the solution pop into your mind without the need for debugging? If not, you're not alone. In this article, I'll introduce a method to provide real-time feedback on the functions you create as you work through a problem. After all, none of us are infallible geniuses who can always nail it first try like ThePrimeagen.

## Table of contents
- [Ok but what is a REPL](#ok-but-what-is-a-repl)
- [Ok but why should I use this?](#ok-but-why-should-i-use-this)
- [A new cycle begins?](#a-new-cycle-begins)
- [How to setup a simple REPL in a ruby application](#how-to-setup-a-simple-repl-in-a-ruby-application)
- [Introducing pry, a more concise implementation](#introducing-pry-a-more-concise-implementation)
	- [Seeing a class documentation](#seeing-a-class-documentation)
	- [How to visualize exceptions](#how-to-visualize-exceptions)
	- [Listing methods and editing them](#listing-methods-and-editing-them)
	- [Reloading the code to keep iterating over it](#reloading-the-code-to-keep-iterating-over-it)
- [Debugging tests with a REPL](#bonus-debugging-tests-with-a-repl)
- [Conclusion](#conclusion)

## Ok but what is a REPL?

[![REPL](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.cherkashin.dev%2Fassets%2Frepl%2Frepl-meme.png&f=1&nofb=1&ipt=a0b043d8684763fff7dd672daf45d42fc02a9c40a3600c7f75401649871675ac&ipo=images)](https://www.cherkashin.dev/assets/repl/repl-meme.png)
A REPL is essentially a continuous loop that takes a command, evaluates it, shows the result, and then repeat the loop by waiting for another command. What makes REPLs handy is their ability to remember the context of the previous command. To see this in action, you can open your shell right now and launch `irb` to experiment with Ruby functions.

```sh
$ irb
irb(main):001> 1 + 1
=> 2
irb(main):002> raise 'Some error'
(irb):2:in `<main>': Some error (RuntimeError)
	from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/lib/ruby/gems/3.2.0/gems/irb-1.8.0/exe/irb:9:in `<top (required)>'
	from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/bin/irb:25:in `load'
	from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/bin/irb:25:in `<main>'
irb(main):003>
```

As you can observe we get the output for simple operation stuff like `1 + 1` or even exception raising, quite cool right? This is an amazing feature because we can quickly evaluate complex operation to help us understand the algorithm as it's being written.

## Ok but why should I use this?

![Pasted image 20230903131823](https://github.com/cherryramatisdev/public_zet/assets/86631177/b34ea38e-812e-4eef-a675-4d8928fe675d)

If you're a Ruby developer, chances are you've used `irb` for basic tasks like calculating `1 + 1` (after all, we're programmers, not mathematicians). But how can we go from using the REPL as a calculator to change the way we write software? To illustrate this, let's consider the following scenario: You're working on an API application that follows a simple service layer architecture and you just started to write a first implementation for a service. Now, what would you do to quickly execute that service and test if your logic is close?

Well, you probably have two options right now:

- *1. You write a test following TDD philosophy*: The TDD philosophy is wide spread across the ruby community and can indeed be very useful, but there is a catch to this (as anything in programming) and it's that most of the test you need to write while developing a feature will be throw away once you finish the service implementation, because tests are useful to validate the business logic and code contracts, not to validate if a syntax is correct.
- *2. You write the rest of the HTTP layer to test using curl*: Most of the times I do this (when coding javascript) just to be able to have a curl command that I can run over and over again while filling my code with logs until I finish the implementation. But again this is not the purpose of the http layer, ideally you'll write the http layer as the last thing on your implementation because it's just a communication part.

On a "REPL Driven Development" scenario you have the exact opposite approach because you start testing your service as soon as you create the file and write a function, this is awesome because we stop thinking about running the tests or testing with curl as a debug step and start being a validation one where you check whether your solution is in sync with the initial purpose (described in tests or documentation).

It's important to emphasize that I'm not against TDD philosophy, I find it quite useful and I want to use tests where they're great, that is checking if my solution goes along with the initial purpose. We shouldn't use tests to check whether our code running.

## A new cycle begins?

![Pasted image 20230903142637](https://github.com/cherryramatisdev/public_zet/assets/86631177/c8df33a4-045f-4ad2-874c-178f340a9956)

When we start considering the REPL as part of our technical experience, another cycle start to merge that *lives alongside the TDD cycle* where we only reach for testing when we have something that run without errors keeping our test cases much more concise and bound to the original purpose. We also don't need to keep writing wrappers that just run our service so we can keep running a `curl` command to check logs.

Unfortunately there are a few languages that implement a usable setup to use the REPL in a application context *as far as I know*, some of them are: ruby, elixir, haskell, clojure (and various other lisps).

Although it's cool to discuss the theory part and to think about a new philosophy, I want to help you put all of this in practice so let's go for the practical goodies shall we?

## How to setup a simple REPL in a ruby application?

On all my application tutorials I start by setting up an application level REPL, it's basically a `console` script that loads all the files inside your project, if you're using a framework like [Ruby on Rails](https://rubyonrails.org) or [Hanami](https://hanamirb.org) you already have a console by running the command `console` also.

To setup it's pretty simple, you just need to create a file inside `bin/console` and require all the files you want to use on a REPL, most of the times we use gems like [zeitwerk](https://github.com/fxn/zeitwerk) to provide the auto requiring, but if you want to do it manually, refer to the example below:

```ruby
#!/usr/bin/env ruby

# Ensure tht the 'lib/' directory is in the load path
$LOAD_PATH.unshift(File.expand_path('../lib', __dir__))

# Load all *.rb files in the lib/ directory
Dir[File.expand_path('../lib/*.rb', __dir__)].each do |file|
  require file
end

require 'irb'
IRB.start
```

If you want a more complex example using auto-requiring and dependency injection, please refer to my article at: https://dev.to/cherryramatis/creating-a-sinatra-api-with-system-wide-dependency-injection-using-dry-system-10mp

## Introducing pry, a more concise implementation

All of my recent tutorials and projects were primarily managed using the default Ruby REPL, `irb`, and I must say it's been nothing short of **amazing**. However, what ultimately prompted me to switch to [Pry](https://pry.github.io) was its offering of **better defaults**. But what exactly does that mean? Let me demonstrate:

### Seeing a class documentation

While running `pry`, you'll find two essential aliases for exploring class details: `$` and `?`.

When you use `?`, you're essentially invoking the `ri` command on a method to view the YARD docs of it. However, `pry` provides a more user-friendly approach. You can use `?` on nearly any class or method without worrying about whether it will function as expected.

```sh
$ bin/console
[1] pry(main)> ? MonadicExceptions::Result.from_exception

From: /Users/cherryramatis/Repos/monadic-exceptions/lib/result.rb:12:
Owner: #<Class:MonadicExceptions::Result>
Visibility: public
Signature: from_exception(callback)
Number of lines: 16

This static method act as a bridge between exception to Result, it
supress any raises a method invokes and transform into a valid Failure
return.
param callback [Proc]
return [Failure({error: Symbol, where: String, orig_exception: Exception, message: String}), Success(data)]

def self.from_exception(callback)
  result = callback.call

  Dry::Monads::Result::Success.new(result)
rescue => e
  exception_name = MonadicExceptions::ResultSanitizer.new.treat_exception_name(e.class.to_s)
  Dry::Monads::Result::Failure.new({ error: exception_name,
                                     where: callback.source_location.first, orig_exception: e,
                                     message: e.message })
end
[2] pry(main)>
```

If you just want to see the source code it's recommended to use `$`:

```sh
[2] pry(main)> $ MonadicExceptions::Result.from_exception

From: /Users/cherryramatis/Repos/monadic-exceptions/lib/result.rb:12:
Owner: #<Class:MonadicExceptions::Result>
Visibility: public
Signature: from_exception(callback)
Number of lines: 10

def self.from_exception(callback)
  result = callback.call

  Dry::Monads::Result::Success.new(result)
rescue => e
  exception_name = MonadicExceptions::ResultSanitizer.new.treat_exception_name(e.class.to_s)
  Dry::Monads::Result::Failure.new({ error: exception_name,
                                     where: callback.source_location.first, orig_exception: e,
                                     message: e.message })
end
[3] pry(main)>
```

### How to visualize exceptions

During the iterative development process, encountering error exceptions is almost inevitable, and it's crucial to be able to visualize and comprehend these errors.

In Pry, you have the command `wtf` (yes, you read it correctly) that allows you to view the details of the last triggered exception:

```sh
[7] pry(main)> MonadicExceptions::Result.method_raising
RuntimeError: A error
from /Users/cherryramatis/Repos/monadic-exceptions/lib/result.rb:29:in `method_raising'
[8] pry(main)> wtf
Exception: RuntimeError: A error
--
0: /Users/cherryramatis/Repos/monadic-exceptions/lib/result.rb:29:in `method_raising'
1: (pry):3:in `__pry__'
2: /Users/cherryramatis/.asdf/installs/ruby/3.2.2/lib/ruby/gems/3.2.0/gems/pry-0.14.2/lib/pry/pry_instance.rb:290:in `eval'
3: /Users/cherryramatis/.asdf/installs/ruby/3.2.2/lib/ruby/gems/3.2.0/gems/pry-0.14.2/lib/pry/pry_instance.rb:290:in `evaluate_ruby'
4: /Users/cherryramatis/.asdf/installs/ruby/3.2.2/lib/ruby/gems/3.2.0/gems/pry-0.14.2/lib/pry/pry_instance.rb:659:in `handle_line'
[9] pry(main)>
```

Bonus point: if you append `?` to the command, it'll show more detail about the exception depending on the number of `?` inserted.

![Pasted image 20230903171910](https://github.com/cherryramatisdev/public_zet/assets/86631177/7f0fbf97-727f-41cd-83a3-4d8138635b7c)

### Listing methods and editing them

I use this **a lot** – it's truly amazing to iterate through code using the REPL as the primary source of truth. You can effortlessly navigate your codebase with context by utilizing commands like `ls`, `cd`, and `edit`.

To list the methods of a class, it's as simple as applying `ls` to it:

```sh
[13] pry(main)> ls MonadicExceptions::Result
MonadicExceptions::Result.methods: from_exception  method_raising
MonadicExceptions::Result#methods: blau  blau2  teste
[14] pry(main)>
```

If you want to look methods from a module or class as entry point you can first `cd` into it:

```sh
[14] pry(main)> cd MonadicExceptions
[15] pry(MonadicExceptions):1> ls
constants: Result  ResultSanitizer
locals: _  __  _dir_  _ex_  _file_  _in_  _out_  pry_instance
[16] pry(MonadicExceptions):1> ls Result
MonadicExceptions::Result.methods: from_exception  method_raising
MonadicExceptions::Result#methods: blau  blau2  teste
[17] pry(MonadicExceptions):1>
```

Lastly but still important, you can `edit` at any time to open your code editor directly on the line:

<a href="https://asciinema.org/a/606116" target="_blank"><img src="https://asciinema.org/a/606116.svg" /></a>
> Disclaimer: Yes it works with code from gem or ruby source code too!

### Reloading the code to keep iterating over it

Here enter the true reason why I prefer pry over irb and is that pry provide a whole ecosystem of gems that enhance the way you use pry. On the matter of "reloading" code I'll present the gem pry reload.

The [pry-reload](https://github.com/runa/pry-reload) gem provide the `reload!` command to smartly reload your changed code so you can keep iterating over the solution without closing and reopening the REPL as you can see below:

<a href="https://asciinema.org/a/606121" target="_blank"><img src="https://asciinema.org/a/606121.svg" /></a>

## Bonus: Debugging tests with a REPL
Alongside the wide range of gems around the pry REPL we have [pry-rescue](https://github.com/ConradIrwin/pry-rescue) that allow us to start a debugging REPL as soon as a test fails, that way we can investigate and fix it before waiting for all the other tests to run:

```sh
$ rescue rspec
From: /home/conrad/0/ruby/pry-rescue/examples/example_spec.rb @ line 9 :

     6:
     7: describe "Float" do
     8:   it "should be able to add" do
 =>  9:     (0.1 + 0.2).should == 0.3
    10:   end
    11: end

RSpec::Expectations::ExpectationNotMetError: expected: 0.3
     got: 0.30000000000000004 (using ==)
[1] pry(main)>
```

On that session you can use commands like `try-again`, `break` and `play` to rerun the test, add a breakpoint or running a specific line or method on the context of the breakpoint.

If you want to see more about using pry as a REPL and more detail on pry-rescue, checkout this conference talk: <https://www.youtube.com/watch?v=D9j_Mf91M0I>

## Conclusion

I hope I've effectively conveyed the idea that utilizing a REPL can be a powerful tool for enhancing your understanding of code. This concept truly amazed me when I explored languages such as Clojure and Elixir. It has significantly boosted my confidence when dealing with complex algorithms because it allows me to visualize the output at each step of the implementation process.

Well that is it for me today, if I can help with anything please leave a comment! May the force be with you 🍒
