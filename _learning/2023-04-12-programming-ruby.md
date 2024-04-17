---
title: Notes on 'Programming Ruby' book
excerpt: The Pragmatic Programmers' Guide
collection:
  - software_engineering
  - learning
tags:
  - ruby
---
Book Link: [https://pragprog.com/titles/ruby5/programming-ruby-3-2-5th-edition/](https://pragprog.com/titles/ruby5/programming-ruby-3-2-5th-edition/)

My takeaways:

# Part 1

## Chapter 2: Ruby.new

Ruby is a pure object oriented language with no basic types.
> all Ruby types are objects, and there are no non-object basic types that behave differently. However, many languages claim to be object-oriented, and those languages often have a different interpretation of what object-oriented means and a different terminology for the concepts they employ.
```ruby
histogram = Hash.new(0) # The default value is zero
regerx_comparator =~ /Ru(by|st)/
```

the braces bind more tightly than the do/end pairs

## Chapter 3: Classes, Objects, and Variables

Whenever you’re designing an Object-Oriented system, a good first step is to identify the domain concepts you’re dealing with.

It’s easy to imagine that the two variables here, @isbn and isbn, are somehow related. It looks like they have the same name, but they don’t. The former is an instance variable, and the “at” sign is actually part of its name.
