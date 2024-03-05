---
title: Complementing exceptions - Introducing monads at ruby
description: Let's learn how to use dry-monads to keep our codebase straightforward and easy to use without exceptions
tags: 'ruby,monads,dry'
cover_image: ''
canonical_url: null
published: false
---

Have you ever considered your approach to handling exceptions? I'm referring to the method in which you use the raise keyword within a class, and then you utilize the rescue keyword in the function that calls that method. However, in contemporary programming languages like OCaml, Rust, Elm, Haskell, and Go, there exists an alternative approach that is contrary to exceptions. Essentially, errors are treated as values, and we manage them as regular variables using constructs like match statements or simple if statements. In this article, we will delve into the implementation of this technique using the dry-monads gem.

## Table of contents

- [What is the problem with exceptions?](#what-is-the-problem-with-exceptions)
- [What are the pros and cons of using errors as values](#what-are-the-pros-and-cons-of-using-errors-as-values)
- [Putting our hands to work](#putting-our-hands-to-work)
- [Conclusion](#conclusion)

## What is the problem with exceptions?

Exceptions can be quite easy to raise when you're just looking at the method you're developing, but when you try to consume a library or a method inside your big codebase it's noticeable some annoyances that we'll look right now:

- *Does this method trigger an exception?* :: Whenever a method is invoked in your codebase or through a third-party library, there's no immediate certainty regarding whether that function will result in an exception being thrown. In the most unfortunate scenario, if you haven't referred to the documentation or examined the code to identify any unhandled exceptions, a sudden issue arises. An unmanaged exception emerges from your application, and probably production is down right now.
- *Which method triggered that error?* :: It's quite common to have a method that consume a lot of other methods, if all of those methods trigger different errors it can be very hard to identify which method triggered what while we're developing the method, error messages can be helpful but not 100% accurate.

## What are the pros and cons of using errors as values?

So far I presented to you the pain points of exceptions and introduced a possible solution (the dry-monad gem), but since in the programming field **there is no silver bullet** it's crucial to understand the pros and cons of every possible solution, and that's exactly what we're going to see on this section below:

### Pros

- *The runtime helps you (a little at least)* :: Imagine the following scenario, you call a function and expect to return a `User`, but that method returns a `Error`, suddenly you can't use `user.name` because ruby will tell you that variable is an error type and not a `User`, leading to easy debugging and preventing future bugs at production.

> Disclaimer: This is not as good as having compile time error, but the idea here is to improve your current development experience by bringing the exceptions *closer* to you.
 
- *Deal with error handling before business logic* :: While this might be a more personal preference than a purely technical advantage, working with errors as values allows you to explore fully the power of early returns by handling all the unhappy paths at the beginning of your method.

- *Clear view of which method returned which error* :: The beauty of dealing with errors as return values becomes evident by the transparency it brings to the codebase. As errors are directly returned from methods, it becomes clear which error corresponds to which method.

### Cons

- *Challenges while managing deeply nested errors* :: As we presented on previous topics the benefit of linking errors to their original methods, a difficulty arises in scenarios where you have a deeply nested method chain, in this case it's hard to keep context of where these exceptions passed through.

## Putting our hands to work
First let's define some disclaimers about [dry-monads](https://dry-rb.org/gems/dry-monads/1.3/):

1. Dry monads is **not** about avoiding exceptions, it's more about using exceptions within a controlled environment where you **know** you're raising a exception.
2. Dry monads are not perfect, ruby is a dynamic language and we can't have perfect compile time checks, but we can improve our development experience at the best we can.

Now that we settled our points, let's create a sample project to present how to use this new gem:

### 1. Creating the project

You can create a sample project with:

```sh
mkdir project && cd project && bundle init
```

And then add the only dependency of this project, [dry-monads](https://dry-rb.org/gems/dry-monads/1.3/):

```sh
bundle add dry-monads
```

### 2. How to return an error

Initially, we will examine the `Result` monad. If you're familiar with Rust, comprehending this concept might be relatively straightforward. Essentially, the Result monad encapsulates two potential outcomes: either a `Success(value)` or a `Failure(error)`.

To work with this result monad, first we need to require the library and then for convenience include the `Dry::Monads[:result]` so we can use `Success` and `Failure` without the module prefixes as showed below:

```ruby
require 'dry/monads/all'

class Auth
  include Dry::Monads[:result]

  # @param name [String]
  # @return [Failure(Symbol), Success({ name: String })]
  def authenticate(name:)
    return Failure(:unauthorized) unless name == 'correct'

    Success({ name: 'cherry' })
  end
end
```

If we try to call this method expecting to see a `name` parameter from it like this:

```ruby
val = Auth.new.authenticate(name: 'incorrect')

puts val.name
```

You'll get an error from the ruby runtime:

```sh
$ ruby main.rb
main.rb:19:in `<main>': undefined method `name' for Failure(:unauthorized):Dry::Monads::Result::Failure (NoMethodError)

puts val.name
        ^^^^^
```

Cool right? Now the runtime help us understand if a function trigger an error
or not, and we can handle it property, but how do we get the object inside the
`Failure` on this and log into the console? Let's see two approaches for that.

### 3. How to unwrap the Result variants

#### 1. Pattern matching

You can use the new pattern matching syntax introduced on [ruby version 2.7](https://ruby-doc.org/core-2.7.2/doc/syntax/pattern_matching_rdoc.html) to unwrap both variants like below:

```ruby
case Auth.new.authenticate(name: 'incorrect')
in Dry::Monads::Result::Success({name: String} => user)
  puts user
in Dry::Monads::Result::Failure(:unauthorized => error)
  puts error
end
```

This pattern will match on the `Failure` variant and the output print will be:

```sh
$ ruby main.rb
:unauthorized
```

If you change the name parameter from "incorrect" to "correct" you'll have the following output print instead:

```sh
$ ruby main.rb
{:name=>"cherry"}
```

#### 2. If statements

Using plain old if statement we'll need to use some new methods
It's also possible to use the good old if statements, the result provide boolean methods and also a bind method to unwrap the variant. On the example below we handle it with a plain `puts`, but you can imagine how easy is to use a early return to handle the failure or success variant cases instead.

```ruby
value = Auth.new.authenticate(name: 'incorrect')

value.bind { |user| puts user } if value.success?
value.bind { |error| puts error } if value.failure?
```

As you can see, we have some methods such as `success?` and `failure?` that return booleans, facilitating our life when dealing with control statements. Additionally, we have the `bind`  designed to unwrap the result variant with a closure.

#### 3. Getter methods

Another way to unwrap a specific variant is to use the correspond getter method, this is specially useful when you're already inside a if statement and can be used as follow:

```ruby
value = Auth.new.authenticate(name: 'incorrect')

puts "The error variant is: #{value.failure}" if value.failure?
puts "The success variant is: #{value.success}" if value.success?
```

#### 4. Yield syntax

The yield syntax is a method to unwrap only the `Success` variant of a result without entering a closure, if the method return a `Failure` the unwrapping will **not** happen, so it's recommended to handle the specific failure cases before using the `yield`.

> Disclaimer: The `include` statement it's required to use the `yield` syntax.

```ruby
class Runner
  include Dry::Monads::Do.for(:call)

  def call
    value = Auth.new.authenticate(name: 'incorrect')

    return value.failure if value.failure?

    yield value
  end
end

puts "Result: #{Runner.new.call}"
```

### 4. Dealing with deep nested errors

At the beginning of this article I presented a problem with handling errors as values. That problem is when you have to return different errors for a deep nested method - where a method invokes another, and so forth. But how do we deal with this problem?

In languages like Golang, a function like `errors.Wrap()` exists to facilitate the contextual addition to an error, simplifying the identification of the error origin and providing a lot more information besides just a error message.

Using `dry-monads`, we can leverage the full power of ruby dynamic nature by allowing us to return anything inside the `Failure` variant, that way we can create complex data structures like hashes to register context about the error call stack.

Let's assume the same class we had before, but with a tweak on the error handling:

```ruby
require 'dry/monads/all'

class Auth
  include Dry::Monads[:result]

  # @param name [String]
  # @return [Failure({ error: Symbol, context: String, username: String }), Success({ name: String })]
  def authenticate(name:)
    return Failure({ error: :unauthorized, context: 'Auth#authenticate', username: name }) unless name == 'correct'

    Success({ name: 'cherry' })
  end
end
```

As you can see we can return an object with some keys that provide more information about that error, where it was called and any useful information about it, that freedom allow us to create a key like `parent: 'ParentClass#parent_method'` that essentially mimics the functionality of `errors.Wrap` in the Golang world. We can for sure create more complex structures with custom classes, but on this article I chose to go with a more simple and straightforward approach to introduce the potential!

### 5. Bonus, dealing with the null representation

We saw how to handle failures and success variants for our business logic, but maybe you're thinking with yourself *"I can abstract the absence of value as well?"* and you would be absolute right we can!

The absence of value can be understand as `None` and the value itself can be understand as `Some`, dry-monads gem provide this amazing functionality for us using the same concepts as we saw with the `Result`:

Consider a similar class as we saw above but using the maybe variants instead of the result ones.

```ruby
require 'dry/monads/all'

class Auth
  include Dry::Monads[:maybe]

  # param name [String]
  # @return [None(), Some({name: String})]
  def authenticate(name:)
    return None() unless name == 'correct'

    Some({ name: 'cherry' })
  end
end

none_val = Auth.new.authenticate(name: 'incorrect')
some_val = Auth.new.authenticate(name: 'correct')

puts "None -> #{none_val}"
puts "Some -> #{some_val}"
```

With this sample code our output will be the following:

```sh
$ ruby main.rb
None -> None
Some -> Some({:name=>"cherry"})
```

Similar to the `Result` monad we can do pretty much every control statement as previously showed, below we'll see all of them briefly:

#### 1. Pattern matching

```ruby
require 'dry/monads/all'

class Auth
  include Dry::Monads[:maybe]

  # param name [String]
  # @return [None(), Some({name: String})]
  def authenticate(name:)
    return None() unless name == 'correct'

    Some({ name: 'cherry' })
  end
end

case Auth.new.authenticate(name: 'correct')
in Dry::Monads::Maybe::None
  puts 'None branch'
in Dry::Monads::Maybe::Some({name: String} => user)
  puts "Some branch #{user}"
end
```

#### 2. If statements

```ruby
require 'dry/monads/all'

class Auth
  include Dry::Monads[:maybe]

  # param name [String]
  # @return [None(), Some({name: String})]
  def authenticate(name:)
    return None() unless name == 'correct'

    Some({ name: 'cherry' })
  end
end

option = Auth.new.authenticate(name: 'incorrect')

puts 'This is the none option' if option.none?
option.bind { |opt| puts "This is the some option #{opt}" } if option.some?
```

It's important to observe that we don't need to use the `bind` method on `None` because this variant will just represent the nothingness of value.

#### 3. Yield syntax

Differently than the `Result` monad, `Maybe` don't provide us a getter method so we need to rely on the yield syntax when we don't want to use a closure like on `bind`.

```ruby
class Runner
  include Dry::Monads::Do.for(:call)
  include Dry::Monads[:maybe]

  def call
    option = Auth.new.get_user_name(id: 1)

    return None if option.none?

    yield option
  end
end

puts "Result: #{Runner.new.call}"
```

> Similar to the `Result`, yield just work on the happy paths (in this case the `Some` variant).

## Conclusion

As always I hope you liked this article and learned something new, I'm working on a new gem to wrapping exceptions and returning this monads, I hope to get something working soon and writing the part 2 of this article. May the force be with you!