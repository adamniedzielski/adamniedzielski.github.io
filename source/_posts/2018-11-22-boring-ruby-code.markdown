---
layout: post
title: "Boring Ruby Code"
date: 2018-11-22 16:38:37 +1100
comments: true
categories:
- ruby
- programming
- talks
- ruby on rails
- software engineering
---

In 2017 I delievered a talk with the title "Boring Ruby Code" at two conferences -
Brighton Ruby Conf and Southeast Ruby. After that I always wanted to write a
blog post that summaries the approach, but I have never gotten around to do that.
Today is the day so here we go.

<!-- more -->

This is just a short summary, not one-to-one transcription of the talk.
You can watch the [video from Southeast Ruby](https://youtu.be/t9x3-6ZQ7xY?t=339).

1. Boring code is easier to understand. In your team you have (or at least should
   have) people at different stages of their career. Certain constructs in Ruby
   do not bring you much value, but make the code harder to understand for
   entry level programmers.
2. Boring code is easier to read. Programmers spend most of their time reading the
   code so it makes sense to optimise for the reading time and not the writing time.
   Less common constructs increase the reading time even for people who understand
   how they work.
3. Boring code is easier to delete. There is no "pride" associatied with boring
   code, so the decision to delete it comes easier.
4. Iterating over an array of names to assign values or define methods is just lazy
   and brings no value.
5. `send` is a code smell (`public_send` too).
6. Dynamically defining methods is a perfect way to prevent other programmers from
   discovering where the method is defined.
7. Dynamically defining methods where method name is dynamically constructed is a
   no-go.
8. Metaprogramming capabilities of Ruby can steer our attention away from standard
   refactoring practices. A lot of code duplication can (and should) be solved by
   extracting a method, not by metaprogramming.
9. `method_missing` is a code smell.
10. "Smart code" can hide a lot of simple mistakes just by looking smart.
    Programmers assume that a piece of code is right, because it looks smart.
11. `is_a?` and `respond_to?` is a code smell.
12. Once you start using `is_a?` and `respond_to?` they tend to leak to multiple
    places in your codebase.
