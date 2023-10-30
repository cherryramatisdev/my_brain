---
title: Metaprogramming in ruby
description: Metaprogramming is the reason why ruby and frameworks like rails feels so magical, let's learn how this magic work and how we can do it by ourselves?
tags: ruby,programming,tutorial,metaprogramming
cover_image: ""
canonical_url: 
published: false
---
The ruby programming language is known for two major facts: One is its core philosophy with object-oriented programming where "Everything is an object", the other important one is its incredible flexibility to define DSLs and "magic" classes. This magic is on purpose and a quite special feature of ruby called **metaprogramming**, in this article we'll see more about the deep nested details of ruby and how to create magic APIs with metaprogramming!

## Table of contents

- [What is metaprogramming anyway?](#what-is-metaprogramming-anyway)
- [In ruby everything is an object, what does that mean?](#in-ruby-everything-is-an-object-what-does-that-mean)
- [But what about rails? How this framework applies that concept for maximum developer experience](#but-what-about-rails-how-this-framework-applies-that-concept-for-maximum-developer-experience)
- [How to define methods dynamically](#how-to-define-methods-dynamically)
- [Using hooks to detect moments on the instantiation of the class](#using-hooks-to-detect-moments-on-the-instantiation-of-the-class)
- [Conclusion](#conclusion)

## What is metaprogramming anyway?

Metaprogramming is a concept where you write a program to write a program, confusing, huh? But trust me it's quite simple, imagine the following scenario:

You create a function that write another file with a specific functionality, or even append content on the same file with the whole implementation for a class or an object or whatever, this is code creating more code got it? This is metaprogramming!

A cool example of metaprogramming are DSLs (Domain specific languages) that define a specific syntax within the same language to express far more complex concepts, such as querying a database or defining routes. It's not new that the ruby ecosystem uses a lot of metaprogramming, so we can keep writing elegant and clean code with that "magic" feel to it.

## In Ruby everything is an object, what does that mean?

It's possible that you saw the famous phrase "In ruby, everything is an object", but what does that even mean and how it's related to metaprogramming at all?

In ruby all the things are objects under the hood, even the core data structures like numbers, strings, etc. as you can see on the example below:

```ruby
irb(main):009> 5.class
=> Integer
irb(main):010> 5.class.class
=> Class
irb(main):011> ''.class
=> String
irb(main):012> ''.class.class
=> Class
irb(main):013> true.class
=> TrueClass
irb(main):014> true.class.class
=> Class
```

At the end of the day, even simple structures are defined from a common class object and can have all the things objects are known for such as methods, inheritance, etc...

This allows crazy manipulation possibilities, and it's on that specific field where metaprogramming lives and thrives, since all things are objects we gain the power of changing that object dynamically. Allowing us to create DSLs within the language easily.

Let's try to understand this metaprogramming thingy together without using any modern syntax provided by ruby, shall we?

> Disclaimer: The following example are **not** how to actually implement metaprogramming, it's just an abstraction so you can understand the general idea around the concept.

Imagine we have a hash with a function inside, simulating a class object with method, we can even call this function by accessing the "property" and calling the "method".

```ruby
my_obj = {
  fn: ->() { puts 'original method' }
}

my_object[:fn].() # => original method
```

After defining this initial object, it's possible to write a function that receives this object and append another function to it as follows:

```ruby
my_obj = {
  fn: ->() { puts 'original method' }
}

def add_new_method(obj, cb)
  {another_fn: cb}.merge(obj)
end
```

And with this function working, we can append new functionalities to it by calling the newly created function:

```ruby
my_obj = {
  fn: ->() { puts 'original method' }
}

def add_new_method(obj, key, cb)
  {key => cb}.merge(obj)
end

my_obj[:fn].() # => original method

my_obj = add_new_method(my_obj, :another_fn, ->() { puts 'another method' })
my_obj[:another_fn].() # => another method

my_obj = add_new_method(my_obj, :hello, ->(target) { puts "hello #{target}" })
my_obj[:hello].("world") # => hello world
```

This is the core concept of metaprogramming, of course that modern functions provided by ruby itself lets you do far more things such as listen to specific events etc. But the core mindset around it is manipulating the underlying objects and adding new functions to it.

## But what about rails? How this framework applies that concept for maximum developer experience

The [Ruby on Rails](https://rubyonrails.org) framework is the most known and powerful ruby gem for a long time, and its core philosophy evolves around providing the smallest bit of elegant code to achieve a lot of features on your application. To provide that level of abstraction and elegant syntax, rails rely a lot on metaprogramming, so we can write less and achieve more on our codebase.

Methods like `validates`, `has_many`, `scope`, `before_save` are just a couple that comes with the ORM part that provide various benefit for declarative model code as shown on the examples below:

```ruby
class User < ApplicationRecord
  has_many :posts
  validates :username, presence: true
  before_save :hash_password

  scope :recent, -> { where('created_at > ?', 1.week.ago)}

  private

  def hash_password
    self.password = hash(self.password)
  end
end
```

- `has_many` enable us to express a database relationship of users and posts within the model itself, in this particular example the `Post` model should use another method called `belongs_to`.
- `validates` allow validation when we try to perform any creation method using this particular model
- `before_save` act as a hook informing a method as a symbol, this particular method will be called before the record is saved on the database.
- `scope` provide a way to create a new method on this model with the particular query, that way we can call something like `User.recent` and get the result of the function.

And there is much more metaprogramming patterns on different parts of the framework such as controllers, views, helpers, etc. It's truly an important part of what makes rails a beloved framework.

## How to define methods dynamically

Now for the more practical parts, creating methods dynamically is as easy and as elegant as anything in ruby! Hopefully we're using a well though language, right? 👀 */j*

The most basic action is defining a method dynamic, and we can use the built-in syntax provided by ruby called `define_method`, below we'll see a practical example:

```ruby
class FirstExample
  define_method(:example_method) do
    puts 'example method'
  end
end

FirstExample.new.example_method # => example method

class SecondExample
  def self.define_new_method(method_name, &body)
    define_method(method_name, &body)
  end
end

SecondExample.define_new_method(:hello) { |target| puts "Hello #{target}" }
SecondExample.new.hello("world") # => Hello world
```

The `FirstExample` class show how a simple usage of the `define_method` looks like, it accepts a symbol as the first parameter and a block that can optionally receive an argument and perform any actions (it'll be replaced as the class method once it's instantiated), at the end of the example we can simply instantiate the class with `.new` and call the dynamically defined method.

The `SecondExample` is more involved and define a method to define methods(**finally some meta-language references**), although it's simply an abstraction on top of the `define_method` function itself we can see the power that this can lead to a real codebase right? An important detail is that we accept the second parameter as `&body` because this tells ruby to accept as a block that we'll latter use on the example with the `hello` method.

## Using hooks to detect moments on the instantiation of the class

Ruby not only provide a way to create new methods dynamically, but also provide hooks, so we can run arbitrary code when a specific action happen (around the object cycle of course).

Things such as running some code when a class is inherited or included and even when a particular method does not exist within a class, pretty cool and powerful, huh?

We'll see in detail about some possibilities with different hooks below:

### method_missing

This hook function allow the run of arbitrary code once an unknown method is called within a class, below it's an example of a class implementing it:

```ruby
class MethodMissingExample
  def method_missing(name, *args)
    puts "An unknown method was called using the name ->  #{name} and the arguments -> #{args.inspect}"
  end
end

MethodMissingExample.new.test_method(1, 2, 3) # => An unknown method was called using the name ->  test_method and the arguments -> [1, 2, 3]
```

As you can see, by simply creating a method with the name `method_missing` we already enable this functionality and can act on it with the `name` and the `args` of the method at hand. This can be quite useful if you're working with public code that it's used by other developers and want to provide custom experience when a method is misspelled or indeed not implemented.

### included

This method works only for module and will run a particular block when the module is included into another class or module as shown below:

```ruby
module IncludedExample
  def self.included(name)
    puts "The #{name} was included!!"
  end
end

module Test
  include IncludedExample # => The Test was included!!
end
```

Similarly, we have a method for the `extend` and `prepend` action within modules, it works the same as the includes, and it's shown below with an example:

```ruby
module ExtendedExample
  def self.extended(name)
    puts "The #{name} was extended!!"
  end
end

module PrependedExample
  def self.prepended(name)
    puts "The #{name} was prepended!!"
  end
end

module Test
  extend ExtendedExample # => The Test was extended!!
  prepend PrependedExample # => The Test was prepended!!
end
```

### inherited

Back to the classes with a similar functionality, we can create a hook that will be run every time a class is inherited, as shown below:

```ruby
class InheritedExample
  def self.inherited(name)
    puts "The #{name} class was inherited!!"
  end
end

class Test < InheritedExample # => The Test class was inherited!!
end
```

## Conclusion

Hope this was useful for anyone reading it! metaprogramming is a very fun subject to learn and use, it can create very messy code but with good architecture it's possible to create amazing experience such as with Ruby on Rails. Just reach out if I can help with anything and may the force be with you 🍒
