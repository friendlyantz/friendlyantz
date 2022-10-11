---
title: "Book: '99 Bottles' by Sandi Metz"
excerpt: "first book every Ruby dev should read"
collection: learning
---

Repo: [https://github.com/friendlyantz/99bottles](https://github.com/friendlyantz/99bottles)

My takeaways:

- 1. When you were new to programming you wrote simple code. Although you may not have appreciated it at the time, this was a great strength. Since then, you’ve learned new skills, tackled harder problems, and produced increasingly complex solutions. Experience has taught you that most code will someday change, and you’ve begun to craft it in anticipation of that day. Complexity seems both natural and inevitable.

This is the basic promise of Object-Oriented Design (OOD): that if you’re willing to accept increases in the complexity of your code along some dimensions, you’ll be rewarded with decreases in complexity along others. OOD doesn’t claim to be free; it merely asserts that its benefits outweigh its costs.

The price you pay for DRYing out code is that the invoker of the new method no longer knows the result, only the message it should send.

---

Consistency - The style of the conditionals to be consistent. 
Duplication - The code should not duplicate both data and logic. Data duplications are easy to see, while logic is not obvious.
Names -  methods are defined, and messages are sent.

---

1. How difficult was it to write?
2. How hard is it to understand?
3. How expensive will it be to change?

---

Software Engineering is an art in some sense, and you can describe it as easily
as an art object or wine:
“ I like my code to be elegant and efficient. — Bjarne Stroustrup
inventor of C++

“ Clean code is full of crisp abstractions
— Grady Booch
author of Object Oriented Analysis and Design with Applications

“ Clean code was written by someone who cares.
— Michael Feathers

---

Since form follows function, good code can also be defined simply, and somewhat circularly, as that which provides the highest value for the lowest cost.

---

metrics are a crowd-sourced opinions about the quality of code.

---
Chapter 2

The purpose of some of your tests might very well be to prove that they represent bad ideas

---

The distinction between intention and implementation [ ] allows you to understand a computation first in essence and later, if necessary, in detail.
— Kent Beck Implementation Patterns (p. 69)

---

Issue with tests:

Many applications have tests that are difficult to understand, challenging to change, and prohibitively time-consuming to run. Instead of enabling change, these tests actively impede it.


A great deal of this pain originates with tests that are tied too closely to code. When this is true, every improvement to the code breaks the tests, forcing them to change in turn. Therefore, the first step in learning the art of testing is to understand how to write tests that confirm what your code does without any knowledge of how your code does it.

---

Testing, done well, speeds development and lowers costs. Unfortunately it’s also true that flawed tests slow you down and cost you money.

