---
title: Creating a sinatra API with system wide dependency injection using dry-system
description: Let's talk about creating an API using sinatra, rom-rb and dry-system with a great modular architecture!
tags:
  - ruby
  - backend
  - sinatra
  - dry
  - rom
  - article
cover_image: ""
canonical_url: 
published: true
---
## Table of contents
   1. [Introduction](#introduction)

   2. [What do we do when our applications start to grow?](#what-do-we-do-when-our-applications-start-to-grow)

   3. [How do we solve these problems? dry-system to the rescue!](#how-do-we-solve-these-problems-dry-system-to-the-rescue)

   4. [Improving our Sinatra application](#improving-our-sinatra-application)

   5. [Adding dry-system and dry-auto_inject gems as our dependency injection layer](#adding-dry-system-and-dry-auto_inject-gems-as-our-dependency-injection-layer)

   6. [Adding database connections with ROM and our modular architecture](#adding-database-connections-with-rom-and-our-modular-architecture)

   7. [Conclusion](#conclusion)

## Introduction

Today, among beginners with Ruby, it's common to think about two possible
paths when developing an application; if you want a simple single-file API, just use
[Sinatra](https://sinatrarb.com/) and for everything else, use [Ruby on
Rails](https://rubyonrails.org/). Well, in this article, allow me to provide a
way to manage a big application using Sinatra as the HTTP library and dry-rb
libraries as the glue to a modular architecture.

## What do we do when our applications start to grow?

What do we do when an application with a single Ruby file starts to grow with more dependencies? For me personally, the answer is **dependency injection**.
Basically, I start thinking about how I'll manage the configuration of all these new libraries and use them quickly on my routes, so it's trivial to split
routes into services and controller classes in the future.

OK, but how do we do that?

The basic understanding of dependency injection can be seen from the following perspective:

Consider this "service" class:

```ruby
class SomeService
  def something_important
    # doing something important here
    
    'information'
  end
end
```

If we want to inject this service, we can simply instantiate it in our constructor, for example, on a controller:

```ruby
require_relative 'services/some_services'

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

But what problem does this approach have? Well, this doesn't provide much complexity for a small-scale application, and it's pretty simple to keep all the components isolated and available, but we introduce some annoyances for medium to large-scale applications, such as:

1. *Not all providers have simple setups* :: Some providers, like ORMs need more configuration, and this can be hard to maintain and make available through the application.
2. *Some providers depend on another provider* :: It's quite hard to manage by hand when you want one provider for the database connection and another one for the repositories, and this happens a lot.
3. *Require hell* :: On Ruby, we don't have the habit of importing all our libraries and internal code on every single file; frameworks such as Ruby on Rails provide auto-require for files with business logic, and when you roll an application by hand, it's hard to develop without this feature.

## How do we solve these problems? dry-system to the rescue!

We'll assume a simple Sinatra application and evolve from that by adding dry-system to manage our dependencies; later on, we'll even add a persistence layer using a gem called rom-rb to increase the functionality for a more realistic API example.

### A simple Sinatra application

Sinatra is a lightweight library that's quite simple to set up, but let's start with a more structured project, shall we?

> DISCLAIMER: This part assumes basic knowledge about ruby language and the sinatra library.

#### Start a bundle project

```sh
mkdir myproject && cd myproject && bundle init
```

#### Add our gems

```sh
bundle add sinatra puma
```

#### Create a router class to encapsulate our execution

Located at `config/router.rb`

```ruby
require 'sinatra/base'

class Router < Sinatra::Base
  get '/' do
    {message: 'Hello world'}.to_json
  end
end
```

4. Add a `config.ru` to serve as the entry point for our application

Located at `config.ru` on the project root

```ruby
require_relative 'config/router'

Router.run!
```

With this initial setup, we should be able to run the application with `bundle exec puma` and see a JSON as the response.

## Improving our Sinatra application

To make it easier for us to visualize the benefit of dependency injection, let's add some structure to this simple route by creating two simple abstractions: controller and service.

First, we'll create a service located at `lib/service/user.rb` with the following content:

```ruby
module Services
  class User
    def index
      'teste'
    end
  end
end
```

Then let's create a sample controller located at `lib/controllers/user.rb` with the following content:

```ruby
require_relative 'lib/services/user'

module Controllers
  class User
    def initialize
      @service = Services::User.new
    end

    def index
      {message: @service.index}.to_json
    end
  end
end
```

As you can see, we're already instantiating the service the old way, so we can compare by adding `dry-system` to it!

To wrap up, just update the content of `config/router.rb` file:

```ruby
require 'sinatra/base'
require_relative 'lib/controllers/user'

class Router < Sinatra::Base
  get '/' do
    Controllers::User.new.index
  end
end
```

Simple right? Now everything should work fine, but we won't stop there, so let's start integrating `dry-system` into it and seeing the benefits.

## Adding dry-system and dry-auto_inject gems as our dependency injection layer

### Adding our gems

```sh
bundle add dry-system dry-auto_inject zeitwerk
```

### Making our application REPL work

A REPL (Read-Eval-Print Loop) is a very important tool for Ruby developers. Both the Rails and Hanami frameworks provide one, so we'll set up a simple REPL for our application. This will allow us to further integrate the dependency injection layer, which will make our code more modular and easier to test.

To do this, we'll create a file called config/boot.rb and add the following code:

```ruby
ENV['APP_ENV'] ||= 'development'

require 'bundler'
Bundler.setup(:default, ENV.fetch('APP_ENV', nil))
```

After that create a script file under `bin/console` with the following content:

```ruby
#!/usr/bin/env ruby
require 'irb'

IRB.start
```

To make it executable you can run `chmod +x ./bin/console`

Now we should have a working REPL for the application!

### Creating our main container

This container will be used to register all the other components of our application

Create a file under `config/application.rb` with the following:

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

You can see with this code that we're already solving one of the problems; the `component_dirs.add` method and the Zeitwerk instance will automatically require all our code inside the `lib` and `config` folders.

> Note: The zeitwerk gem is doing the lazy loading for us.

Let's include this in our entry points to make it work right away.

On the `config.ru` and on the `bin/console` we'll add the following:

```ruby
require_relative 'config/application'

Application.finalize!
```

The `finalize!` method makes the `Application` instance variable available for the whole application and lazy-loads our files under the `lib` and `config` folders.

> Tip: You can and it's encouraged to remove the `require_relative` from your controller and router file

Now you can run `bin/console` and check the application instance by typing `Application` on the REPL.

### Adding a sample service as a provider

Now that we have our main container, it's just a matter of registering providers to it, just like the following:

Create a file located at `config/providers/services.rb` with the following content:

```ruby
Application.register_provider(:services) do
  start do
    register('services.user', Services::User.new)
  end
end
```

And after creating this provider, we'll load it on our entry point files; these are the only places where we'll require files.

On `config.ru`:

```ruby
require_relative 'config/providers/services'
```

And on `bin/console`:

```ruby
require_relative '../config/providers/services'
```

### Enjoying the benefits of our work

Going back to our controller class, we can rewrite it like this:

```ruby
module Controllers
  class User
    def initialize
      @service = Application['services.user']
    end

    def index
      {message: @service.index}.to_json
    end
  end
```

See how the controller class doesn't know anything about which class it's getting from `Application['services.user']`? This is so cool because if you want to change your service completely, you can simply change the class instantiation on the provider file.

This initial purpose already works for us, right? But we'll keep going further.

## Adding database connections with ROM and our modular architecture

Now that we have a basic understanding of how `dry-system` works to modularize our application, let's add a database layer using this knowledge while levering `rom-rb` with it.

### Adding our gems

```sh
bundle add rom rom-repository rom-sql pg
```

### Registering a database connection as a provider for our system

Since we're already using `dry-system` up to this point, let's work with it by adding the database connection as a provider:

Create a file located at `config/providers/db.rb` with the following content

> Disclaimer: This assumes you're running a PostgreSQL database.

```ruby
Application.register_provider(:db) do
  prepare do
    require 'rom'
    require 'rom-sql'
  end

  start do
    connection = Sequel.connect('postgres://postgres:postgres@localhost:5432/example_database', extensions: %i[pg_timestamptz])
    register('db.connection', connection)
    register('db.config', ROM::Configuration.new(:sql, connection))
  end
end
```

As you can see, the `register_provider` method provides a simple DSL that we can use to isolate our whole setup by requiring the correct libraries on `prepare` and then instantiating or registering the objects on `start`.

### Adding support for migration commands

Now that we have our base connection done, let's create a `Rakefile` on the root of our project with the following content:

```ruby
require 'rom-sql'
require 'rom/sql/rake_task'

require_relative 'config/boot'
require_relative 'config/application'
require_relative 'config/providers/db'

namespace :db do
  task :setup do
    Application.start(:db)
    config = Application['db.config']
    config.gateways[:default].use_logger(Logger.new($stdout))
  end
end
```

As you can see, we can use the `start` method as an alternative to the `finalize!` that injects all our providers. That way, we only enable the database layer through the `:db` symbol. This allows us to inject on the `:setup` task.

Now we should be able to run the following command:

```sh
rake "db:create_migration[create_users]"
```

This should create a file located at `db/migrate/3128932189_create_users.rb`, on this file, we can complement the following DSL to create a sample table for our application:

```ruby
ROM::SQL.migration do
  change do
    create_table :users do
      primary_key :id
      column :name, String
      column :email, String
    end
  end
end
```

And finally, by running the following command, we can persist this migration on the Postgres database:

```sh
rake db:migrate
```

### Defining our relations and repositories

In the `rom-rb` gem, we define our main classes as **relations** and **repositories**. Relations mimic the structure of our Postgres table, while repositories define our actions on that relation.

First, we'll define a relation to represent the new table we created. To do this, we'll create a file called `lib/relations/users.rb` and add the following code:

```ruby
module Relations
  class Users < ROM::Relation[:sql]
    schema(:users) do
      attribute :id, Types::Integer
      attribute :name, Types::String
      attribute :email, Types::String

      primary_key :id
    end
  end
end
```

Here we're using the simple DSL provided by `ROM::Relation` class to mimic our migration with the correct types for each attribute.

Now for the repository, we can create a file at `lib/repos/user.rb` with the following content:

```ruby
require 'rom-repository'

module Repos
  class User < ROM::Repository[:users]
    commands :create

    # @param limit Integer
    def all(limit = 10)
      users.limit(limit).to_a
    end
  end
end
```

Repositories on ROM have [sample commands](https://rom-rb.org/4.0/learn/core/commands/) for common actions, such as 
creating, updating, and deleting records. However, for more complex 
queries, we need to write our own methods. In this case, I have provided
 a simple `all` method that returns all the users, limited to a certain number.

### Making our code available through the codebase

Since we defined two new components for our applications, we'll create two new providers on the system.

First, let's create a provider at `config/providers/persistence.rb` with the following content:

```ruby
Application.register_provider(:persistence) do
  start do
    target.start :db

    config = target['db.config']

    config.register_relation(Relations::Users)

    register('container', ROM.container(config))
  end
end
```

Similar to our `Rakefile` we're using the `start` method to make the `db` provider available when we're instantiating our relation class.

Then let's create another provider at `config/providers/repos.rb` with the following content:

```ruby
Application.register_provider(:repos) do
  start do
    target.start :persistence

    register('repos.user', Repos::User.new(target['container']))
  end
end
```

See how we `start` the persistence provider we defined previously ? We don't need to start the `db` provider because dry-system will go to the persistence provider and start there, so we can have as many co-dependent providers as we want.

Since we added new providers, we'll update our entry points files as usual:

At `config.ru`:

```ruby
require_relative 'config/providers/persistence'
require_relative 'config/providers/repos'
```

And at `bin/console`:

```ruby
require_relative '../config/providers/persistence'
require_relative '../config/providers/repos'
```

### Refactor time, shall we?

Now that we've defined our required providers, it's just a matter of using them on the layer we want; this layer will be the service class for us.

On the service class at `lib/services/user.rb`, we'll rewrite to use the repository:

```ruby
module Services
  class User
    def initialize
      @repo = Application['repos.user']
    end

    def list_all
      users = @repo.all

      users.map do |user|
        { id: user.id, name: user.name, email: user.email }
      end.to_json
    end
  end
end
```

The refactor is done! Pretty easy, right? Our route should now use the database to provide a list of users.

## Conclusion

I hope this article is useful for anyone who ends up reading it. I tried to demonstrate how easy it is to decouple application parts and manage them, even when each part requires complex setups.

Furthermore, I'm always available to help with any doubt or just to chat about cool Ruby stuff. May the force be with you!

