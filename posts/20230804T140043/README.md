---
title: Creating a sinatra API with system wide dependency injection using dry-system
description: Let's talk about creating an API using sinatra, rom-rb and dry-system with a great modular architecture!
tags: 'ruby,backend,sinatra,dry,rom'
cover_image: ''
canonical_url: null
published: false
---

## Introduction

Today among the beginners with ruby it's common to think about two possible
paths, if you want a simple single file API just use
[Sinatra](https://sinatrarb.com/) and for everything else use [Ruby on
Rails](https://rubyonrails.org/). Well, on this article allow me to provide a
way to manage a big application using Sinatra as the http library and dry-rb
libraries as the glue to a modular architecture.

## 1. What we do when our application start to grow?

When your application with a single Sinatra file start to grow with more
dependencies what we do? for me personally it's **dependency injection**,
basically I start thinking about how I'll manage the configuration of all these new libraries and use it quickly on my routes, that way it's trivial to split
routes into services and controllers classes on the future.

OK but how we do that?

The basic understanding of dependency injection can be view from the following perspective:

Consider this "service" class:

```ruby
class SomeService
  def something_important
    # doing something important here
    
    'information'
  end
end
```

If we want to inject this service we can simply instantiate it on our constructor, for example on a controller:
require_relative 'services/some_services'

```ruby
class SomeController
  def initialize
    @service = SomeService.new
  end

  def index
    response = @service.something_important

    {result: response}.to_json
  end
end
```

But what problem this approach has? Well, this for small scale applications don't provide much complexity and it's pretty simple to keep all the components isolated and available, but we bring some annoyances for medium to large scale applications such as:

1. *Not all providers have simple setups* :: Some providers like ORMs need more configuration and this can be hard to maintain available through the application
2. *Some providers depend on another provider* :: This happen a lot, like when you want a provider for the database connection and another for the repositories and it's quite hard to manage this by hand.
3. *Require hell* :: On ruby we don't have the habit of importing all our libraries and internal code on every single file, frameworks such as Ruby on Rails provide auto require for files with business logic and when you roll a application by hand it's hard to develop without this.

## 2. How we solve these problems? dry-system to the rescue!

We'll assume a simple sinatra application and evolve from that by adding dry-system to manage our dependencies, on the and we'll even add a persistence layer using rom-rb just to avoid having a simple API with local variable as return.

### A simple Sinatra application

Sinatra is a lightweight library that it's quite simple to setup, but let's start with a more structured project shall we?

1. Start a bundle project

```sh
mkdir myproject && cd myproject && bundle init
```

2. Add our gems

```sh
bundle add sinatra puma
```

3. Create a router class to encapsulate our execution

Located at `config/router.rb`

```ruby
require 'sinatra/base'

class Router < Sinatra::Base
  get '/' do
    {message: 'Hello world'}.to_json
  end
end
```

4. Add a `config.ru` to serve as the entrypoint for our application

Located at `config.ru` on the project root

```ruby
require_relative 'config/router'

Router.run!
```

With this initial setup we should be able to run the application with `bundle exec puma` and see a json as the response.

### Let's improve by adding dry-system to it

Now we can start defining our containerization approach to our app with dry-system and dry-auto_inject

#### 1. So first we'll add the required libraries

```sh
bundle add dry-system dry-auto_inject zeitwerk
```

#### 2. Still before adding the specific gem config, let's setup the basics to have a REPL support for our application

Create a file `config/boot.rb` with the following content:

```ruby
ENV['APP_ENV'] ||= 'development'

require 'bundler'
Bundler.setup(:default, ENV.fetch('APP_ENV', nil))
```

After that create a script under `bin/console` with the following content:

```ruby
#!/usr/bin/env ruby
require 'irb'

IRB.start
```

To make it executable you can run `chmod +x ./bin/console`

Now we should have a working REPL for the application!

#### 3. Let's create our main container

This container will be used to register all the other components of our application

Create a file under `config/application.rb` with the following

```ruby
require 'dry/system'

class Application < Dry::System::Container
  configure do |config|
    config.root = Pathname('.')

    config.component_dirs.loader = Dry::System::Loader::Autoloading

    config.component_dirs.add 'lib'
    config.component_dirs.add 'config'
  end
end

loader = Zeitwerk::Loader.new
loader.push_dir(Application.config.root.join('lib').realpath)
loader.push_dir(Application.config.root.join('config').realpath)
loader.setup
```

You can realize with this code that we're solving one of the problems already, the `component_dirs.add` and the zeitwerk instance will auto require all our code inside the `lib` and `config` folder.

> The zeitwerk gem is doing the lazy loading for us.

Let's include this on our entry points to make it work right away

On the `config.ru` we'll add the following:

```ruby
require 'config/application'

Application.finalize!
```

And on our console located at `bin/console`:

```ruby
require 'config/application'

Application.finalize!
```
The `finalize!` method does all the requires and makes the `Application` instance variable available for the whole application.

Now you can run `bin/console` and check the application instance by typing `Application` on the REPL.

#### 4. Let's add a sample service as provider
