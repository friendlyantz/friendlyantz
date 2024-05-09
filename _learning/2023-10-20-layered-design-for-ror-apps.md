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

# Chapter 13

Infra level parts:
• Database adapters
• Third-party API clients
• Caching and storage systems (that is, Active Storage backends)
• Configuration providers (credentials, secrets, and so on)
• Background processing engines (for example, Sidekiq and GoodJob)
• Web servers (for example, Puma and Unicorn) and Rack middleware
• Logging and monitoring tools

The **abstraction distance** (the number of in-between abstractions before we reach the actual implementation) can be big in Rails.

 > implementations are not owned by our application or the framework, while infrastructure abstractions are. i.e.
 > ActiveRecord::Base.connection - still not the actual database connection wrapper (which is database-specific) but, instead, an infrastructure abstraction provided by the framework.
