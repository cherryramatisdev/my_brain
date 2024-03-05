---
title: What can we learn with functional language OOP vs Functional
description: The more time goes by, the more I become a functional programming enthusiast. With this article we'll discuss concepts around functional programming to learn in a simple way and propose those concepts to be used even in OOP land!
tags: programming,beginners,functional
cover_image:
canonical_url:
published: false
---

The more time goes by, the more I become a functional programming enthusiast. Even when I'm working in an OOP codebase, I try to apply small concepts aimed at simplifying the code and predicting outcomes more easily. As a Rubyist, I also love how functional code is trivial to write unit tests.

The goal of this article is to share my views on functional programming concepts and propose an approach where we can use functional concepts within your already written OOP code. Hopefully, we'll stop arguing about which paradigm is better and start composing good ideas to produce increasingly better code each time!

## Table of Contents

- [What is OOP anyway](#what-is-oop-anyway)
- [What is functional anyway](#what-is-functional-anyway)
- [What is a class and how can we think about it differently](#what-is-a-class-and-how-can-we-think-about-it-differently)
- [To mutate or not to mutate: What is immutability](#to-mutate-or-not-to-mutate-what-is-immutability)
- [The monster under the bed: what are side effects](#the-monster-under-the-bed-what-are-side-effects)
- [Isolate everything: what are pure functions](#isolate-everything-what-are-pure-functions)
- [Using functional patterns without going full haskell](#using-functional-patterns-without-going-full-haskell)
- [Conclusion](#conclusion)


## What is OOP anyway

OOP, or better known as Object-Oriented Programming, is a paradigm dating back to 1967. Its greatest representatives are Simula, Smalltalk, and Java. The thought process behind it is to reduce the amount of "global" state by enforcing encapsulation practices to group those states and any behavior modifying them under a common "entity" or "object."

In fact, the name "Object-Oriented Programming" has been widely discussed over the years. One of the creators of OOP, Alan Key, actually wanted to focus more on the messaging aspect of the paradigm. This means that we should emphasize encapsulation and allow communication of state and behavior between objects. Perhaps in a different universe, we could have had "Messaging-Oriented Programming." However, the name OOP has endured over the years, and here we are!

I don't know about you, but this simple process of considering another possible name for the paradigm made my mind go crazy, rethinking some concepts and actually simplifying the way I architect my software.

## What is functional anyway

The functional programming paradigm emerged in 1958 with the advent of the first Lisp language (good old days). Its roots can be traced back to the lambda calculus by Alonzo Church. The core principle of functional programming revolves around minimizing the dependence on state within the codebase.

In contrast to Object-oriented Programming (OOP), which allows state but emphasizes encapsulation, functional programmers strive to prioritize writing components that are stateless. This approach encourages the creation of code that is independent of external state variables.

Furthermore, even when state is introduced, it is essential to write it thinking about immutability, purity of functions and even avoiding side effects. All this concepts will be covered further as the article goes.

## What is a class and how can we think about it differently

I think everyone has had that classic lecture where we were taught about a class "Animal" that includes a class "Dog," right? You probably heard those same words (at least I did).

> A class is a blueprint for an entity in the real world, describing its characteristics and actions.

While not incorrect, I would like to suggest a slight change in word usage to help clarify the encapsulations for better understanding, as it did for me. Let's consider the following new quote:

> A class is a way to encapsulate state and behaviors that operate on that state.

This simple word change really made a twist in my mind. I stopped trying to think of classes as real-world entities and started to think simply as another way to group state with similar context together and expose functions to operate on those states. I hope this slight change also helps you review your own concepts!

## To mutate or not to mutate: What is immutability

Now we're reaching the first real functional concept, and a very important one, I may add (everything is planned here)! To understand immutability properly, let's review the ways we deal with values in programming:

Typically, we bind values to variables so we can operate on them later, right? Something as simple as

```ruby
# Bounding values to variables
name = 'Cherry'
age = 23

# Operating on it and bounding to another variable
year_born = Time.now.year - age

# Printing it
puts "#{name} has #{age} years old and was born at #{year_born}"
```

With these variables, it's quite common to make changes to the original variable by updating its values, but the missing piece here is: modifying a variable is a **destructive** action.

But... why? Well, let's imagine multiple operations (functions or blocks of code) modifying the same variable at different moments and frequencies. With this situation, we create a lot of problems such as:

-   **1. Inability to reorder the operations or change them at all:** When we have so much dependent code, it's even hard to reorder or change the code because everything is bound to a specific order of changes.

-   **2. Mental overhead to understand what the code is doing:** Although this is a personal opinion, I think it's a widely agreeable one. Highly mutable code can easily become messy and cryptic to understand the data flow, requiring tools such as a debugger to step through the transformations.

-   **3. Difficulties while testing:** Mocking a particular state of the function transformations is really hard, and this will gradually expand your unit tests until they aren't units anymore.

Immutability can be defined as a practice to avoid changing (or mutating) any variable inside our program. Although depending on the language, we may need to make concessions and mutate some controlled variables, the overall lesson to be learned here is:

> We should avoid mutating variables with no defined scope at all costs.

With this phrase, I mean that it's okay to create a scoped variable inside a function and mutate it there. However, as soon as you pass this mutable variable to another function, you will increase the number of targets mutating the same variable, and your control will slowly be lost. This is the exact situation that we want to avoid!

## The monster under the bed: what are side effects

This topic generates a lot of heat whenever someone raises it for discussion. I may not cover every nuance of this subject, but I will definitely explain to you what they are and how I manage side effects in my own software, okay?

Well then, side effects are every computation that interacts with outside resources (or "outside world") by calling a protocol (HTTP, WebSocket, GraphQL, etc.) or even manipulating stdin/stdout. Yeah, I know, even our harmless `print` gets the blame here. 😔

But different from mutability, instead of avoiding the usage as much as possible, we should isolate it in specific functions that deal with the side effect alone. This way, we separate our code into "functions that do not perform any side effects" and "functions that perform side effects." But why worry about this separation at all?

Every time we trigger any action to the "outside world," we lose control over what can happen for this particular piece of computation (like when performing an HTTP call, the server may be down or may not exist at all). Other problems include the difficulty of testing and a reduction in the predictability of the code.

Since we can't write any real-world software without side effects, the general advice is to clusterize it into small functions that handle it separately with a proper abstraction around errors. That way, it's possible to test only the function that we control 100% and mock all the functions that perform side effects.

For example, consider the following function that performs an HTTP request and small functions that transform the data returned from it.

```ruby
require 'faraday'

module MyServiceModule
  # This function perform side effects
  def perform_http_request
    conn = Faraday.new(url: "fakeapi.com")

    begin
      response = conn.get
      {ok: true, data: response.body}
    rescue  => e
      {ok: false, error: e}
    end
  end

  # These functions doens't perform any side effects
  def upcase_name(name)
    return '' unless name.is_a?(String)

    name.upcase
  end

  def retrieve_born_year(age)
    return 0 unless age.is_a?(Integer)

    Time.now.year - age
  end
end
```

> See how I said about a "abstraction around error" ? This is exactly what was implemented on the code example above, instead of letting the exception bubble up our code we abstracted against a Hash.

After defining these functions with clear definition between "side effects" and  "no side effects" it's really easy to predict what will happen in our code and also easier to test as you can see below:

```ruby
require 'minitest/autorun'

class TestingStuff < Minitest::Test
  def test_upcase_name
    assert_equal MyServiceModule.upcase_name "cherry", "CHERRY"
    assert_equal MyServiceModule.upcase_name "kalane", "KALANE"
    assert_equal MyServiceModule.upcase_name "Thales", "THALES"
  end

  def test_retrieve_born_year
    Time.stub :now, Time.new(2024, 3, 5) do
      assert_equal MyServiceModule.retrieve_born_year 23, 2001
      assert_equal MyServiceModule.retrieve_born_year 20, 2004
      assert_equal MyServiceModule.retrieve_born_year 14, 2010
    end
  end
end
```

This strategy is really great because you don't even need to worry about the side effect part when testing, just write assertions for the transformation part of the code that actually do something and you'll end up with better tests that actually validate the important part of your codebase! Neat right?

## Isolate everything: what are pure functions

Now it's time to wrap up all the knowledge gained so far. In the previous example, we observed code separated into "side effects" and "no side effects." We also saw how these functions are easier to test and that our main transformation business logic should be kept isolated. Are you wondering what these functions are called? They are **pure functions**!

Let's examine a proper formal definition of pure functions and explore the concept step by step.

> Pure functions are functions that respect immutability, do not perform any side effects, and return the same output given the same parameters.

Basically, pure functions respect all the principles we previously mentioned, plus they always produce the same return for the same parameters. Let's take a look at our previous functions.

```ruby
def upcase_name(name)
  return '' unless name.is_a?(String)

  name.upcase
end

upcase_name('cherry') # => Will be *always* CHERRY
```

With pure functions, we can easily define multiple assertions because we aren't bound to any context that requires extensive mocking. We simply pass the required parameters with static values, and that's it!

Since pure functions are very small and composable, their number increases very quickly. To handle this, functional languages like Elixir provide composition operators like the pipeline, which make it really easy to execute multiple pure functions in sequence.

```elixir
"cherry  "
|> trim
|> upcase # => "CHERRY"
```

> The pipeline operator originates from functions like Bash. You can read more about it here: [<https://dev.to/cherryramatis/linux-filters-how-to-streamline-text-like-a-boss-2dp4#what-is-a-pipeline>]


## Using functional patterns without going full haskell

I always feared learning the functional paradigm because the community made it look really complicated by using ready-made sentences and big concepts for everyone trying to learn some small tips. After grasping many functional languages and trying to learn as much as I can, my goal became to simplify those concepts and, most importantly, advocate for using functional concepts even in your OOP code.

Applying pure functions (or pure methods, if you prefer), immutability, and separation of side effects can make your OOP code look a lot cleaner and decoupled. You don't need to know what a monad is or how to write a compiler by hand in Haskell; you can simply stick with your Ruby on Rails using simple and effective functional concepts!

I hope that with this small article (and the ones that will come in this series) you can improve your codebase with composability and simplicity, no matter which language and framework you choose.

## Conclusion

This article is my attempt to democratize the knowledge of the functional paradigm (within my capabilities and expertise). It's important to emphasize that I'm not a functional expert, and this article is targeted at beginners who know OOP and have an interest in functional programming. I hope it's useful, and I'm willing to help with whatever is needed. May the force be with you 🍒
