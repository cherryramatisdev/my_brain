---
title: Never deal with exceptions again! Introducing monads at ruby
description: Let's learn how to use dry-monads to keep our codebase straightforward and easy to use without exceptions
tags: 'ruby,monads,dry'
cover_image: ''
canonical_url: null
published: false
---

Have you ever considered your approach to handling exceptions? I'm referring to
the method in which you use the raise keyword within a class, and then you
utilize the rescue keyword in the function that calls that method. However, in
contemporary programming languages like Ocaml, Rust, Elm, Haskell, and Go, there
exists an alternative approach that is contrary to exceptions. Essentially,
errors are treated as values, and we manage them as regular variables using
constructs like match statements or simple if statements. In this article, we
will delve into the implementation of this technique using the dry-monads gem.

## What is the problem with exceptions?

Exceptions can be quite easy to raise when you're just looking at the method you're developing, but when you try to consume a library or a method inside your big codebase it's noticeable some annoyances:

- *Does this method trigger an exception?* :: When you make a method call within
your codebase or from an external library, there's no immediate certainty
regarding whether that function will result in an exception being thrown. In the
most unfortunate scenario, if you haven't referred to the documentation or
examined the code to identify any unhandled exceptions, a sudden issue arises.
An unmanaged exception emerges from your application and probably production is down right now.

## What are the pros and cons of using errors as values?

Ok I'm presenting to you a solution that is using errors as values and dealing with them uing regular if statements, but what are the cons for that? Well let me tell you:

### Pros

- *The runtime helps you (a little)* :: If you expect one method to return a `User` and that method return a `Error`, suddenly you can't use `user.name` because ruby will tell you that is a error and it can be easier to discover bugs before deploying to production.

> Disclaimer: I know this isn't as good as having a compiler to tell you that the specific type `Error` don't have `name` as a property, but we don't have this functionality right now (as far as I know).

- *You can view the error handling before your actual logic* :: This is
something I always liked about using early returns for example, with that
pattern you can isolate your unhappy paths at the beginning of functions,
leaving the rest for actual business logic and happy paths.

### Cons

- *It's quite hard to manage deeply nested errors* :: This is probably the only
annoyance you'll get by using errors as values, if you have a method that calls
a method that calls a method and all of them return a error, it can be pretty
hard to differentiate where that error came from.

## Let's put our hands to work

Ok, now that I presented to you my points let's look at how this can be implemented.

We'll create a sample project just with some classes to know how we manage each case.

### 1. Creating the project

You can create a sample project with:

```sh
mkdir project && cd project && bundle init
```

And then let's add the only dependency of this project, [dry-monads](https://dry-rb.org/gems/dry-monads/1.3/):

```sh
bundle add dry-monads
```

### 2. How to return an error

Initially, we will examine the `Result` monad. If you're familiar with Rust,
comprehending this concept might be relatively straightforward. Essentially, the
Result monad encapsulates two potential outcomes: either a `Success(value)` or a
`Failure(:error)`.

Consider the following class with a sample method returning a error:

```ruby
require 'dry-monads'

class SampleClass
  include Dry::Monads[:result]

  def find_user_by_id
    user = nil

    if user.nil?
      Failure(:user_not_found)
    else
      Success(user)
    end
  end
end
```

If you try to call this method excepting to see a `name` parameter from it like this:

```ruby
val = SampleClass.new.find_user_by_id

puts val.name
```

You'll get an error from the ruby runtime:

```
main.rb:19:in `<main>': undefined method `name' for Failure(:user_not_found):Dry::Monads::Result::Failure (NoMethodError)

puts val.name
        ^^^^^
```

Cool right? you can
TODO