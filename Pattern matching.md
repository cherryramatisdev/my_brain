---
title: Pattern matching - Dealing with the if statement nightmare
description: Let's learn how to deal with the if statement nightmare by leveraging better pattern matching patterns both in Ruby and in Elixir.
tags: ruby,elixir,beginners,programming
cover_image: ""
canonical_url: 
published: false
---

I think we all faced(or wrote) that 20 lines of if statements to check a single variable in all the possibilities, and I think we all suffered to add one more statement to that function leaving it even more unreadable. Well, on this article we'll see a better way **in my opinion** called pattern matching, essentially we'll learn how to use `switch cases with steroids`!!!

## Table of contents

- [What is pattern matching and why do we need it?](#what-is-pattern-matching-and-why-do-we-need-it)
- [Elixir pattern matching, where things get mind blowing](#elixir-pattern-matching-where-things-get-mind-blowing)
- [Using pattern matching Ruby vs Elixir](#using-pattern-matching-ruby-vs-elixir)
- [Conclusion](#conclusion)

## What is pattern matching and why do we need it?

`TODO: nao gostei do tom desse paragrafo` In general terms, pattern matching can be understood as a way to express any equality between two terms. We use **simple pattern matching** on basically all programming languages with trivial structures like `if statements` or for some languages we maybe use the concept of `destructuring`. By all means, all those examples are express the same idea about pattern matching: it's a form of expressing equalities, the topic on this article will be specifically on why do we need to learn anything more than that? Why do we need to improve our pattern matching techniques? Isn't if statements enough? I think it's not enough, and I'll try to convince (and hopefully teach) how amazing Ruby and Elixir implement those powerful patterns matching new structures.

For clearance I'll exemplify the initial examples mentioned on the paragraph above so we can move forward together ok? First the simpler form of pattern matching probably is the if statement, that can be expressed this way in ruby:

```ruby
if 1 == 2
  puts 'OMG!!!'
end ```

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

This is **really** cool in my opinion because this is where we'll really see the equality example, see how the array has three items and we're presenting with three "slots" on the left hand side? This make the expression mathematically correct and the ruby interpreter understands perfectly that we're trying to assign each one of those array item with each one of those "slots".

On ruby 3 we can even use the pattern matching idea to validate data structures, cool right? It was introduced the `=>` operator that essentially provide runtime check for a specific pattern so you can use as a layer of validation, just like shown below:

```ruby
irb(main):003:0> {a: 1,b: 2,c: 3} => {a: '1', b: '2', c: '3'}
(irb):3:in `<main>': {:a=>1, :b=>2, :c=>3}: "1" === 1 does not return true (NoMatchingPatternError)
        from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/lib/ruby/gems/3.2.0/gems/irb-1.7.4/exe/irb:9:in `<top (required)>'
        from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/bin/irb:25:in `load'
        from /Users/cherryramatis/.asdf/installs/ruby/3.2.2/bin/irb:25:in `<main>'
irb(main):004:0> {a: 1,b: 2,c: 3} => {a: 1, b: 2, c: 3}
=> nil
```

Cool right? I'm not going to deep down on these specifics about ruby 3 destructuring but I highly suggest you to read all about here: <https://www.fullstackruby.dev/ruby-3-fundamentals/2021/01/06/everything-you-need-to-know-about-destructuring-in-ruby-3/>

## Elixir pattern matching, where things get mind blowing

Until here I've been talking a lot about ruby (my love language) but this article is also about elixir, but... why? Well, elixir bring to the table a whole lot of possibilities because [Jose Valim](https://github.com/josevalim) got the pattern matching concept and basically made an entire language around it, it's crazy and we'll see more about these crazy ideas together.
