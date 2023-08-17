---
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - ruby
  - exercism
---

# Useful Resources

- [ ] [Using Zeitwerk Outside Rails](https://www.akshaykhot.com/using-zeitwerk-outside-rails/) - Zeitwerk is a thread-safe Ruby code loader supporting both autoloading and eager loading. It’s commonly associated with Rails, but can be used without it too.

## Rails Resources

- [ ] [Advanced Active Record Concepts](https://rubyhero.dev/advanced-active-record) - A tour of concepts including locking records to avoid conflicts, using UUIDs as primary keys, fulltext search, using database views, and working with geospatial data. I suspect it might end up with a 2024 update for vector similarity queries.. 😁
- [ ] [Gemfile of Dreams](https://evilmartians.com/chronicles/gemfile-of-dreams-libraries-we-use-to-build-rails-apps) - The Libraries Evil Martians Use to Build Rails Apps
- [ ] [Split your database seeds.rb by Rails environment](https://railsnotes.xyz/blog/split-seeds-rb-by-rails-environment)

# Tools

- [ ] [An OpenAI API client.](https://github.com/alexrudall/ruby-openai)
- [ ] [twilio-ruby](https://github.com/twilio/twilio-ruby)
- [ ] [Bullet Train Rails Template](https://github.com/bullet-train-co/bullet_train)
- [ ] [rails-brotli-cache](https://github.com/pawurb/rails-brotli-cache)
- [ ] [rspec-sidekiq](https://github.com/wspurgin/rspec-sidekiq)
- [ ] [deep_pluck](https://github.com/khiav223577/deep_pluck) - Pluck deeply into nested associations without loading a bunch of records.


# Exercism takeaways

[GitHub exercism/ruby solutions with some comments in commit](https://github.com/friendlyantz/exercism/tree/master/ruby)
Interesting exercises:
- Two Fer
- Resistor Color Duo

### `class` vs `instance` methods

The class level method is there for convenience only, and it should stay inflexible, as it is there for convenience not power. We will see this decision in use in our "expansion", below.

### Hash freezeing and I18N connection

The languages available and the translations available should be a single thing, a Hash. Otherwise there more of a chance that things can become disconnected in the way that they are now related, (but not connected).

### `then` as a circuit breaker
```ruby
# meets condition, no-op
1.then.detect(&:odd?)            # => 1
# does not meet condition, drop value
2.then.detect(&:odd?)            # => nil
```
