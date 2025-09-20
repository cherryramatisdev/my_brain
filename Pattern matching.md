---
title: Pattern matching - Dealing with the if statement nightmare
description: Let's learn how to deal with the if statement nightmare by leveraging better pattern matching patterns both in Ruby and in Elixir.
tags:
  - ruby
  - elixir
  - beginners
  - programming
  - article
cover_image: ""
canonical_url: 
published: true
---

I think we all faced(or wrote) that 20 lines of if statements to check a single variable in all the possibilities, and I think we all suffered to add one more statement to that function leaving it even more unreadable. Well, on this article we'll see a better way **in my opinion** called pattern matching, essentially we'll learn how to use `switch cases with steroids`!!!

## Table of contents

- [What is pattern matching and why do we need it?](#what-is-pattern-matching-and-why-do-we-need-it)
- [How ruby provide good pattern matching](#how-ruby-provide-good-pattern-matching)
  - [Case in, switch cases with steroids](#case-in-switch-cases-with-steroids)
  - [Validation operator, one line to validate them all](#validation-operator-one-line-to-validate-them-all)
- [Elixir pattern matching, where things get mind blowing](#elixir-pattern-matching-where-things-get-mind-blowing)
  - [Basic assignments](#basic-assignments)
  - [Function parameters matching](#function-parameters-matching)
- [Conclusion](#conclusion)

## What is pattern matching and why do we need it?

In general terms, pattern matching serves as a mechanism to express any equality between two terms. We frequently employ basic pattern matching in various programming languages such as the straightforward `if statements` or more specific ones like `destructuring`. However, these examples all share the same fundamental concept: pattern matching is a means of expressing equivalences.

This article aims to explore why it's essential to expand our understanding of pattern matching beyond these elementary concepts. Are `if statements` sufficient on their own? I believe they fall short, and I intend to illustrate the compelling advantages of advanced pattern matching techniques as implemented in languages like Ruby and Elixir. Through this exploration, I hope to convince readers about the transformative power of these new pattern matching structures.

For clearance I'll exemplify the initial examples mentioned on the paragraph above so we can move forward together ok? First the simpler form of pattern matching probably is the if statement, that can be expressed this way in ruby:

```ruby
if 1 == 2
  puts 'OMG!!!'
end 
```

Observe that the equality expression can be viewed at the `1 == 2`, in this specific case will be always false.

Other important form of pattern matching can be found at `destructuring` objects, we can do it as follow in ruby:

```ruby
irb(main):001:0> a,b,c = [1,2,3]
=> [1, 2, 3]
irb(main):002:0> a
=> 1
irb(main):003:0> b
=> 2
irb(main):004:0> c
=> 3
irb(main):005:0>
```

This is seriously awesome, if you ask me! This is where it become evident all those *equality* examples. Check it out: see how the array has three items and we're presenting with three "slots"(a,b,c) on the left-hand side? This makes the expression mathematically correct, so the ruby interpreter is able to understand perfectly our intentions and correctly assign each one of the array itens to each one of those "slots" (as shown on the rest of the REPL example).

## How ruby provide good pattern matching

We've just explored Ruby's capabilities in terms of advanced techniques like destructuring, but let's explore more about pattern-matching. Starting with Ruby 2.7, the language brought a significant amount of features aimed at improving pattern-matching-style coding. These additions, including the `case..in` statement and the introduction of the validation operator, are inspired by the influence of Elixir (that we'll see more about).

### Case in, switch cases with steroids

Switch cases are a pretty common pattern at every language right? most of the times they're viewed as a different way of writing if statements, because of that ruby added pattern matching support with the `in` clause inside a switch case statement, allowing far more complex interaction as shown on the example below:

```ruby
def check_object(obj)
  case obj
  in String => str
    puts "It's a string: #{str}"
  in Integer => num
    puts "It's an integer: #{num}"
  in Array => [first, *rest]
    puts "It's an array with the first element #{first} and the rest #{rest.inspect}"
  in Hash => { key: k, value: v }
    puts "It's a hash with key '#{k}' and value '#{v}'"
  else
    puts "It's something else."
  end
end
```

As you can see we can use the `in` clause to match a specific variable with both type checking and destructuring in the same statement! This makes it really easy to control the state in a clean and direct way through our program.

### Validation operator, one line to validate them all

Although pattern matching is pretty useful for the if statement like approach, we can also use for simpler validation layers. Since ruby 3 we got the possibility to use the same operator `=>` presented at switch cases outside the scope of this statement, allowing for runtime checks that serve as validation points. This allow us developers to simplify all those validation classes and gems by using a simple and concise line with pattern matching as shown below:

```ruby
irb(main):003:0> {a: 1,b: 2,c: 3} => {a: '1', b: '2', c: '3'}
(irb):3:in `<main>': {:a=>1, :b=>2, :c=>3}: "1" === 1 does not return true (NoMatchingPatternError)
        from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/lib/ruby/gems/3.2.0/gems/irb-1.7.4/exe/irb:9:in `<top (required)>'
        from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/bin/irb:25:in `load'
        from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/bin/irb:25:in `<main>'
irb(main):004:0> {a: 1,b: 2,c: 3} => {a: 1, b: 2, c: 3}
=> nil
```

In the first example, we get a runtime error check because the left-hand side `{a: 1, b: 2, c: 3}` didn't match the right-hand side `{a: '1', b: '2', c: '3'}`. This opens a whole set of possibilities. It's simply a matter of changing the right-hand side with a `params` object from Rails and validating at will using a simple line; Ruby is indeed awesome!

Btw, if you want to dig deep down on these specifics about Ruby 3 destructuring and new operators, I highly suggest reading all about it here: <https://www.fullstackruby.dev/ruby-3-fundamentals/2021/01/06/everything-you-need-to-know-about-destructuring-in-ruby-3/>

## Elixir pattern matching, where things get mind blowing

Up to this point, I've dedicated a significant portion of this discussion to Ruby, my beloved language. However, this article isn't just about Ruby; it's also about Elixir. But you might be wondering. Why Elixir? Well, Elixir brings to the table a lot of possibilities thanks to [Jose Valim](https://github.com/josevalim), who incorporated the concept of pattern matching into the entire language. This may seem strange at first, but we'll explore more about those crazy ideas.

### Basic assignments
	
Since Elixir is a language built on top of pattern matching as a mental model, every assignment involves a process of pattern matching through equality. This is super powerful because advanced concepts like destructuring become trivial while using Elixir since it's just a normal assignment. For example:

```elixir
iex(1)> {a, b} = {1,2}
{1, 2}
iex(2)> a
1
iex(3)> b
2
```

See how destructuring turns out to be as simple as creating a new variable? We just expect from one side a tuple of "slots" `{a,b}` and on the other side a tuple of concrete values `{1,2}`, since their formats provide a match, elixir can properly fill out the slots, turning them into variables.

To further prove the point, if we try to break the equality by adding one more element on any side, the runtime will error on us about this misunderstanding, but again without any special operators:

```elixir
iex(1)> {a,b} = {1,2,3}
** (MatchError) no match of right hand side value: {1, 2, 3}
    (stdlib 5.0.2) erl_eval.erl:498: :erl_eval.expr/6
    iex:4: (file)
```

```elixir
iex(1)> {a,b,c} = {1,2}
** (MatchError) no match of right hand side value: {1, 2}
    (stdlib 5.0.2) erl_eval.erl:498: :erl_eval.expr/6
    iex:4: (file)
```

### Function parameters matching

Another possibility provided by the language is to define pattern matching on the parameters for functions with the same name. It may seem like a weird feature, but it turns out to be a really cool feature of the language and an interesting possibility for designing new architectural decisions. Basically, you can define a function with the same name multiple times, and Elixir will use pattern matching to decide which one to execute each time based on the parameters.

For example, imagine we want to define a calculator module. The first thing that crosses your mind is to define a method for each possible action, right? But allow me to provide another way of thinking with the example below:

```elixir
defmodule Math do
  def calc(:sum, num1, num2) do
    num1 + num2
  end

  def calc(:subtract, num1, num2) do
    num1 - num2
  end

  def calc(:divide, num1, num2) do
    num1 / num2
  end
  
  def calc(:multiply, num1, num2) do
    num1 * num2
  end

  def calc(_, _, _) do
    {:error, "Action not implemented"}
  end
end
```

See? By thinking around pattern matching, we could design a far more simple API with possibilities for gradually adding new functionalities. With this proposal, we can simply call the sample method name `calc` and expect a first atom argument that will define which actions it's necessary to take with the following two numbers. Observe even that we can define a default match returning a not implemented error with the last `calc` definition, this allows us to gradually define new actions without worrying about not providing a cool experience for anyone using this module.

## Conclusion

The primary goal of this article was to offer an engaging **introduction** to pattern matching in Ruby and Elixir. I firmly believe that both communities have much to gain from each other, and my journey into learning Elixir has been incredibly fun. I hope that this article has been helpful to anyone reading it. May the force be with you! 🍒
