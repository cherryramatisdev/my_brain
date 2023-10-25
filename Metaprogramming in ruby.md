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

A cool example of metaprogramming are DSLs (Domain specific languages) that define a specific syntax within the same language to express far more complex concepts, such as querying a database or defining routes. It's not new that the ruby ecossystem uses a lot of metaprogramming so we can keep writing elegant and clean code with that "magit" feel to it.

## In Ruby everything is an object, what does that mean?

It's possible that you saw the famous phrase "In ruby, everything is an object", but what does that even mean and how it's related to metaprogramming at all?

## But what about rails? How this framework applies that concept for maximum developer experience
## How to define methods dynamically
## Using hooks to detect moments on the instantiation of the class
## Classes are just objects, how can we dynamically manipulate them?
## Conclusion
