---
# title: "Vi Everywhere"
# permalink: /life/pages/
excerpt: "a place for my `sh` scripts and tricks I discover"
# last_modified_at: 2016-11-03T11:13:12-04:00
collection: learning

# TODO not sure if categories and tags are supported outside of `_posts` dir
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
---

* Table of contents (do not remove this line)
{:toc}

# take an argument from a return value

```sh
your_script | grep keyword_in_line | awk {print $1} | command_to_use_return_value_from_the_previous_step

# Background jobs / Vi sesssion / etc

you can send current session into the background by `ctrl +z` (at least in `VI`)
use `fg` to list all jobs in the background, or switch into a job(tab after `fg`)

## grab args
```sh
jobs -p | ag lala | awk '{print $3}' | xargs kill -9
```