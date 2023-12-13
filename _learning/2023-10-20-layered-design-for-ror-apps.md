---
title: Notes on 'Layered Design for RoR Apps' book
excerpt: notes, useful gems and tricks
collection:
  - software_engineering
  - learning
tags:
  - ruby
---
# Chapter 1


# Chapter 2


# Chapter 3

Adapter vs Plugin vs Wrapper patterns
>  The key difference between adapters and plugins is that plugins provide additional functionality, not just an expected interface.
>  The wrapper pattern could be seen as a degenerate case of the adapter pattern. With a wrapper object, we have both an application-level interface and the implementation encapsulation. Wrappers are usually much easier to deal with in tests than implementations.

# Chapter 4 - Rails Anti-Patterns

- Callbacks, callbacks everywhere
- Concerning Rails concerns
- On global and current states
> can quickly turn into anti-patterns, since they tend to cross the boundaries between layers and lead to code that attracts many responsibilities

# Chapter 5

# Chapter 6