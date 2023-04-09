---
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - docker
---

* Table of contents (do not remove this line)
{:toc}

# Exercism takeaways

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
