---
title: Metaprogramming in ruby
description: ..
tags: ruby,programming,tutorial
cover_image: ""
canonical_url: 
published: false
---
> TODO: Review this paragraph, it's repetitive
Leitura: https://dev.to/daviducolo/metaprogramming-with-ruby-93c

The ruby programming language is known for two major facts: One is it's core philosophy with object oriented programming where "Everything is a object", the other important one is it's incredible flexibility to define DSLs and "magic" classes. This magic is on purpose and a quite special feature of ruby called **metaprogramming**, at this article we'll see more about the deep nested details of ruby and how to create magic APIs with metaprogramming!

## Table of contents

- [What is metaprogramming anyway?](#what-is-metaprogramming-anyway)
- [In ruby everything is an object, what does that mean?](#in-ruby-everything-is-an-object-what-does-that-mean)
- [But what about rails? How this framework applies that concept for maximum developer experience](#but-what-about-rails-how-this-framework-applies-that-concept-for-maximum-developer-experience)
- [How to define methods dynamically](#how-to-define-methods-dynamically)
- [Using hooks to detect moments on the instantiation of the class](#using-hooks-to-detect-moments-on-the-instantiation-of-the-class)
- [Classes are just objects, how can we dynamically manipulate them?](#classes-are-just-objects-how-can-we-dynamically-manipulate-them)
- [Conclusion](#conclusion)

## What is metaprogramming anyway?

Metaprogramming is a concept where you write a program to write a program, confusing huh? But trust me it's quite simple, imagine the following scenario:

You create a function that write another file with a specific functionality, or even append content on the same file with the whole implementation for a class or a object or whatever, this is code creating more code got it? This is metaprogramming!

A cool example of metaprogramming are DSLs (Domain specific languages) that define a specific syntax within the same language to express far more complex concepts, such as querying a database or defining routes. It's not new that the ruby ecosystem uses a lot of metaprogramming so we can keep writing elegant and clean code with that "magit" feel to it.

## In Ruby everything is an object, what does that mean?

It's possible that you saw the famous phrase "In ruby, everything is an object", but what does that even mean and how it's related to metaprogramming at all?

In ruby all the things are objects under the hood, even the core data structures like numbers, strings, etc as you can see on the example below:

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

At the end of the day, even simple structures are defined from a common class object and can have all the things objects are known for such as methods, inheritance, etc..

This allows crazy manipulation possibilities and it's on that specific field where metaprogramming lives and thrives, since all things are objects we gain the power of changing that object dynamically. Allowing us to create DSLs within the language easily.

Let's try to understand this metaprogramming thingy together without using any modern syntax provided by ruby shall we?

Imagine we have an hash with a function inside, simulating a class object with method:

```ruby
my_obj = {
  fn: ->() { puts 'original method' }
}
```

Now that we have this original object, we can write a function that takes this object and append another function to it as follows:

```ruby
my_obj = {
  fn: ->() { puts 'original method' }
}

def add_new_method(obj, cb)
  {another_fn: cb}.merge(obj)
end
```

And with this function put, it's possible to manipulate the original object by appending new functionalities to it:

```ruby
my_obj = {
  fn: ->() { puts 'original method' }
}

def add_new_method(obj, cb)
  {another_fn: cb}.merge(obj)
end

my_obj[:fn].() #=> original method
my_obj = add_new_method(my_obj, ->() { puts 'another method' })
my_obj[:another_fn].() #=> another method
```

This is the core concept of metaprogramming, of course that modern functions provided by ruby let's you do far more things such as listen to specific events and etc, but the core mindset around it is manipulating the underlying objects and adding new functions to it.

## But what about rails? How this framework applies that concept for maximum developer experience
## How to define methods dynamically
## Using hooks to detect moments on the instantiation of the class
## Classes are just objects, how can we dynamically manipulate them?
## Conclusion
