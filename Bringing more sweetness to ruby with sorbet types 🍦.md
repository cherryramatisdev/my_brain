---
title: Bringing more sweetness to ruby with sorbet types 🍦
description: Ever wanted to add type checking for your ruby code? Go from a rubber duck to a full armed duck? So let's learn more about types in ruby using sorbet
tags: ruby,types
cover_image: 
canonical_url:
published: false
---

Have you ever wanted to add type-checking to your Ruby code? Going from a rubber duck to a full-armed duck? (pun intended) no? So allow me to introduce you to type checking using the provided gem [sorbet](https://sorbet.org) created by [Stripe](https://stripe.com/en-gb-br).

## Table of contents

- [Why do we need types at all?](#why-do-we-need-types-at-all)
- [How sorbet aims to introduce types?](#how-sorbet-aims-to-introduce-types)
  - [What is gradual typing?](#ok-but-what-is-gradual-typing)
- [What about ruby 3?](#what-about-ruby-3)
- [Editor support and configuration](#editor-support-and-configuration)
- [Getting our hands dirty! How to create a sample API with sorbet](#getting-our-hands-dirty-how-to-create-a-sample-api-with-sorbet)
  - [Creating a new project and initializing sorbet](#creating-a-new-project-and-initializing-sorbet) - [Defining our main router file](#defining-our-main-router-file)
  - [Creating a service](#creating-a-service)
- [Conclusion](#conclusion)

## Why do we need types at all?

![type check](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fi.imgflip.com%2F207hgd.jpg&f=1&nofb=1&ipt=0692242eb00dd0079a58045ab579a889de9628fafe7fc5f0c2c19d0b82f2d352&ipo=images )

If you have been in the Ruby community for the past couple of years, it's possible that you're not a super fan of types or that this concept never passed through your mind, and **that's totally cool**. I myself love the dynamic and meta-programming nature of Ruby, and honestly, by the time of this article's writing, we aren't on the level of [OCaml](https://ocaml.org) for type checking and inference, but still, there are a couple of nice things that types with sorbet bring to the table:

- *1. Writing less tests*: Ok, this is a sensible topic for us TDD lovers, but writing less tests is not about abandoning tests all together, it is more about *writing tests that actually matter* by testing our business logic and important parts of our application, not by checking if the function `a` receives a number.
- *2. Submitting code with more confidence*: Throw your stones if you Ruby developer didn't get a `'mymethod': undefined method '-' for "test":String (NoMethodError)` on production or on the PR pipeline (if your team likes to have weekends). The whole idea of designing proofs for your code is to solve exactly that situation, because now we have a type check step that ensures this type of bug *most of the time*.
- *3. Trusting more on your environment*: The Ruby community learned through the years to rely on "dumb code analysis tools" such as [Ctags](https://ctags.sourceforge.net), [Grep](https://www.gnu.org/software/grep/) and [find](https://www.gnu.org/software/findutils/) instead of smart code analysis tools (let's be honest, Solargraph isn't that great), but by bringing type proof to our code, it makes it possible to create better tools that analyze our code and provide "go to definition", "completion", "hover", etc. (a.k.a. sorbet LSP).

This piece is a personal opinion, but I firmly believe that embracing type systems is the right choice for modern software development. Perhaps, in the future, our cherished Ruby code will adhere to the principle of "If it compiles, it works," like the outside world (Haskell, OCaml, Elm, etc.).

## How sorbet aims to introduce types?

First let's introduce the tool: [Sorbet](https://sorbet.org) is a gem developed by [Stripe](https://stripe.com) that aims to bring type notation syntax and type checking support for the Ruby ecosystem by utilizing the "Gradual typing" philosophy, it also provide type generation from YARD comments via the [tapioca](https://github.com/Shopify/tapioca) gem, allowing to grow alongside the already built Ruby codebase.

### Ok but what is gradual typing?

Gradual typing is a term that defines a type system that coexists with the idea of `untyped`, where untyped is a part of the code that the compiler needs to ignore at some level (such as `any` for typescript and `mixed` for PHP). This is necessary when developing a type system on top of dynamic languages because it's impossible to just throw away all the previously written code to allow type checking.

Sorbet also goes beyond the `untyped` approach by allowing the developer to **enable type checking per file**. That way, you can absolutely control the areas where you want to type check and the level of strictness you want, as shown below:

- *ignore*: By adding the comment `# typed: ignore` to your file, you tell Sorbet to absolutely ignore that file and its possible errors. It's obvious that this is not recommended at all, right?
- *false*: This is the default state sorbet assumes even if your file doesn't have any comments, and it only reports errors on the syntax (like when you type `deff` instead of `def`).
- *true*: Here is where the fun begins, adding `# typed: true` to your file enables full type checking, but assumes `T.untyped` for all code that doesn't have the `sig` annotation defined.
- *strict*: The strict mode disables the `untyped` for created code and enforces the type notation on the whole file. The general tip is: "If your file works with typed: strict, just let it be".
- *strong*: While the previous strict mode allowed you to explicitly tell that a function is `untyped`, in the strong mode you can't even do that and everything should have a proper type. This mode is cool for experimentation, but I wouldn't use it on production code.

## What about ruby 3?

Recently, we saw Matz talking about the new [RBS](https://www.ruby-lang.org/en/news/2020/09/25/ruby-3-0-0-preview1-released/) solution that should bring type checking to Ruby, and although this is super cool, it's quite new, and the approach has some problems *in my opinion*, such as:

- *1. Lack of LSP*: Since this new type check solution is quite new (at the time of writing), we don't have nice editor support via LSP. Things like [steep](https://github.com/soutaro/steep) will probably solve this in the future, but it's not a reliable solution now. On the other hand, Sorbet has existed for many years on the market and already provides a lot of tools for code intelligence, you can see more in [this blog post](https://sorbet.org/blog/2020/07/30/ruby-3-rbs-sorbet).
- *2. No support for inline types*: The Ruby 3 type system forces you to define the type proof on a separate `rbs` file, this solves part of the problem of typing classes and functions but makes it quite impossible to type declared variables (something that you probably want to do). Sorbet allows such things via the `T.let` function.
- *3. Missing type generation tools*: Differently from the sorbet solution that provides the `tapioca` gem to generate types from YARD docs, allowing the gradual typing of your code base, the Ruby `rbs` format doesn't provide any official solution on that matter and makes the life of early adopters quite difficult.

## Editor support and configuration

![Vim meme](https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fcdn.thenewstack.io%2Fmedia%2F2022%2F08%2F0ae25624-exit-vim-the-arrival-way-6n632sipjag61-1024x692.jpg&f=1&nofb=1&ipt=ee620055da038ebdd6d5bb0cd8c1d8db892873cd9e150ffb1edceb7c63f23770&ipo=images)

Now that I've (hopefully) explained my arguments in favor of sorbet to you, reader, let's look into more practical points, such as the editor support while using sorbet on the codebase. The basic thing to understand is that the same gem used for type checking can be used for hosting an LSP server by passing the `--lsp` flag to it.

- The standard editor for web people (unfortunately); VS Code has official support via the [extension](https://marketplace.visualstudio.com/items?itemName=sorbet.sorbet-vscode-extension), and it's even more detailed on this [official blog post](https://sorbet.org/docs/vscode).
- More heavyweight IDEs like RubyMine also provide official support for sorbet with a [extension](https://www.jetbrains.com/help/ruby/sorbet.html).
- Also my personal and beloved choice; (neo)vim provides support by using any LSP plugin, the main ones also provide official support, such as [nvim lspconfig](https://github.com/neovim/nvim-lspconfig/blob/master/doc/server_configurations.md#sorbet) or [vim ale](https://github.com/dense-analysis/ale/blob/master/doc/ale-ruby.txt#L193).

> Disclaimer: I didn't find any support for TextMate or sublime, sorry :(
## Getting our hands dirty! How to create a sample API with sorbet

Enough with the chatter; it's time to dive into some code. Let's code a simple API using Sinatra to both demonstrate the potential benefits and challenges of achieving type correctness with Sorbet.

> Info: I'll be using a simplified version of the architecture presented on my previous article, if you want to know with more context just look at: https://dev.to/cherryramatis/creating-a-sinatra-api-with-system-wide-dependency-injection-using-dry-system-10mp

### Creating a new project and initializing sorbet

Since this part is well known in all my articles, let's speed run a basic Ruby project setup using Sinatra and Zeitwerk for auto-requiring code:

To begin, let's create a new Ruby project with the necessary dependencies as shown below:

> Disclaimer: The sorbet dependencies will be in a dedicated part

```sh
$ mkdir myproject && cd myproject
$ bundle init
$ bundle add zeitwerk sinatra puma pry pry-reload
```

With our basic dependencies already installed, it's time to configure Zeitwerk to handle the automatic requirement of our files. To do this, create a file at `config/application.rb` and populate it with the following content:

```ruby
# frozen_string_literal: true

require 'zeitwerk'
require 'pathname'

root = Pathname('.')

loader = Zeitwerk::Loader.new

loader.push_dir(root.join('lib').realpath)
loader.push_dir(root.join('config').realpath)

loader.setup
```

We'll enhance the project with basic REPL (Read-Eval-Print Loop) support using **Pry**. To implement this, create a file at `bin/console` and include the following content:

```ruby
#!/usr/bin/env ruby

require 'sorbet-runtime'
require_relative '../config/application'

require 'pry-reload'
require 'pry'
Pry.start
```

> Disclaimer: The inclusion of the sorbet-runtime here allow us to properly run our code with all the type notation that we'll see further.

> Tip: Don't forget to run chmod +x ./bin/console to make your script executable.

After configuring the necessary things for any ruby project let's work on our sorbet environment, start by opening the `Gemfile` file and adding the following content:

```ruby
gem 'sorbet-runtime'

group :development do
  gem 'sorbet'
  gem 'tapioca', require: false
end
```

In this configuration, we're placing both `sorbet` and `tapioca` within the development group. This is because in production, we will exclude type checking (as it should only run on your local machine). However, `sorbet-runtime` is included in all environments to facilitate running code with the specific type notation syntax.

To proceed the setup, let's initialize the sorbet environment with the following commands:

```sh
$ bundle exec tapioca init
```

> Disclaimer: Don't add the sorbet/ folder to your .gitignore file, it's important to keep under version control because you can edit those files or create new ones using the `*.rbi` syntax.

To generate the required type files for our installed gems, run the following command at your shell:

```sh
$ bundle exec tapioca gems
```

To test your installation and see if it's working as it should, you can run this command at your shell:

```sh
$ bundle exec srb tc
```

You should see the message `No errors! Great job.` displayed at your [stdout](https://dev.to/cherryramatis/linux-filters-how-to-streamline-text-like-a-boss-2dp4#what-is-stdin-and-stdout), as a signal that our setup work until this point has been successfully completed 🍒.

### Defining our main router file

Since we want to build a basic API, let's define a router class located at `config/router.rb` with the following content: 

*Notice that we're using the `# typed: true` comment, so we should be getting all the type goodness from the LSP!*

```ruby
# frozen_string_literal: true
# typed: true

require 'json'
require 'sinatra/base'

class Router < Sinatra::Base
  get '/' do
    JSON.dump({ message: 'Hello World' })
  end
end
```

With our routes defined, we need a basic server, right? That's where Puma comes in to save the day! To initialize the Puma server, we'll create a `config.ru` file on the root of our project with the following content:

```ruby
# frozen_string_literal: true

require 'sorbet-runtime'
require_relative 'config/application'

Router.run!
```

### Creating a service

You might be thinking to yourself, "That's all well and good, but where's the type checking?" Not to worry! I've got you covered. Let's create a new file under `lib/services/hello_world_service.rb` with the following content:

```ruby
# frozen_string_literal: true
# typed: true

module Services
  class HelloWorldService
    extend T::Sig

    sig { params(name: T.nilable(String), lang: T.nilable(String)).returns(T::Hash[Symbol, String]) }
    def call(name, lang)
      predicate = case lang
                  when 'pt'
                    'Ola'
                  else
                    'Hello'
                  end

      return { message: "#{predicate} anon" } if name.nil?

      { message: "#{predicate} #{name}" }
    end
  end
end
```

Let's gooo now we're talking about! Isn't this syntax just fantastic? Personally, I find it quite appealing.

To enable the `sig` syntax, we begin by extending the `T::Sig` module within our class. As you can see, there are several options available to us when it comes to describing the type correctness of a method using the `sig` syntax, as demonstrated below:

```ruby
sig { params().returns() }
sig { returns() }
```

You know what best part is? This is 100% valid Ruby syntax! That means you don't require any special syntax highlighters or transpilers – just good old-fashioned and beautiful Ruby code.

In addition to the built-in types provided by the language, such as String and Integer, we have a range of constructs available under the `T` identifier to express more complex types. Here are some key ones that, in my opinion, are crucial to be aware of:

- [T.nilable](https://sorbet.org/docs/tstruct#optional-properties-tnilable-default-and-factory): This tells the type checker that a value can be `nil` and will further error you out to add a guard using the `nil?` method or by convincing the checker to trust you with `T.must`.
  - [T.must](https://sorbet.org/docs/type-assertions#tmust): If you can't or don't want to add an if-clause, you can enforce to the compiler that something is not nil by using the `T.must` syntax like so: `T.must(something_not_nilable)`, this is more like a "trust me" statement.
- [T::Hash](https://sorbet.org/docs/stdlib-generics): This allows us to express the type for a hash by providing both key and value types, like so: `T::Hash[String, Integer]`
- [T::Array](https://sorbet.org/docs/stdlib-generics): The same thing as hash but for arrays, can be used like: `T::Array[String]`.
- [T.let](https://sorbet.org/docs/type-assertions#tlet): This is where Sorbet shines, in my humble opinion, because it enables inline typing for variables like: `foo = T.let(nil, T.nilable(String))`.

Back to our code, since we're receiving both parameters `name` and `lang` on our method, we'll need to modify our router at `config/router.rb`:

```ruby
# frozen_string_literal: true
# typed: true

require 'json'
require 'sinatra/base'

class Router < Sinatra::Base
  get '/:name/:lang' do
    return JSON.dump({ error: 'Provide correct params' }) if params.nil?

    response = Services::HelloWorldService.new.call(T.must(params)['name'], T.must(params)['lang'])

    JSON.dump(response)
  end
end
```

All good, right? But if you run the command `bundle exec srb tc` right now, you'll get the following error:

```
config/router.rb:9: Method params does not exist on T.class_of(Router) https://srb.help/7003
```

Since `params` is a special variable provided by the Sinatra gem, sorbet couldn't infer any type and gave us an error, but don't worry! Let's introduce a cool concept that you'll use a lot while using Sorbet, called `shims`.

Shims are `rbi` files that we declare to provide the types where sorbet couldn't infer by itself. With that in mind let's create a file located at `sorbet/rbi/shims/sinatra_base.rbi` with the following content:

> This is why you need to keep the sorbet/ folder under version control!

```ruby
# typed: true

module Sinatra
  class Base
    extend T::Sig

    sig { returns(T.nilable(T::Hash[String, String])) }
    def self.params; end
  end
end
```

Now we're declaring the `params` as a possibly nullable hash with both key and value as Strings, like this:

```ruby
{"name" => "Cherry Ramatis", "lang" => "pt"}
```

You can observe that on the router file we're using the `T.must` to tell the checker that although it's possible for `params` to return a nil, at the time of running the route it'll not be possible to have any nil value. 

But why didn't we add something like `return if params.nil?` you may be thinking to yourself. In this particular case it's not possible because `params` is a method and not a variable, of course, you can adjust the shim type or even create a variable receiving `params`, but in this case, I preferred to just use the `T.must` clause!

Last but not least we have our **extremely complex** API working:

<a href="https://asciinema.org/a/608555" target="_blank"><img src="https://asciinema.org/a/608555.svg" /></a>

## Conclusion

I hope you can get something useful from this comprehensive guide to sorbet. My main goal was to provide an introduction to this whole ecosystem of new ideas and hopefully encourage more people to try and create the bright future of type checking in Ruby!

You can get more information about the more advanced possibilities of Sorbet `sig` syntax for type notation from [this documentation](https://sorbet.org/docs/sigs). May the force be with you!