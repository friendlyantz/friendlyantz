---
title: "SHell"
permalink: /SHell/
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
  - shell
  - sh
  - bash
  - scripting
---

* Table of contents (do not remove this line)
{:toc}
# Background jobs / Vi sesssion / etc

you can send current session into the background by `ctrl +z` (at least in `VI`)
use `fg` to list all jobs in the background, or switch into a job(tab after `fg`)

# take an argument from a return value

```sh
your_script | grep keyword_in_line | awk '{print $1}' | command_to_use_return_value_from_the_previous_step

## grab args
```sh
jobs -p | ag lala | awk '{print $3}' | xargs kill -9
# or
ps aux | grep ruby | awk '{ print $2 }'
```

## read 
```sh
man signal
```


```sh
echo foo$(you_script)bar
```

---

## Learn Data Streams `stdin` `stdout` `stderr`

[Great Tutorial](https://www.youtube.com/watch?app=desktop&v=zMKacHGuIHI)

```sh
# stdin 0<
# stdout 1> # OR JUST stdout >
# stderr 2>
```

```sh
find /qwerty -type f 1> ./lala 2> ./err
# OR to APPEND instead of overwriting
find /qwerty -type f 1>> ./lala 2>> ./err
```

---

```
chmod 666
```

In Unix-based systems, file permissions are represented by a series of three numbers, each representing a different group of users: the owner, the group, and everyone else. 

The "chmod" command stands for "change mode," and "666" is a code that represents the permissions being granted.

> Remember only, 1-execute, 2-write, 4-read. The sum of these will result in appropriate permission level

    0: No permissions
    1: Execute permission
    2: Write permission
    3: Write and execute permissions
    4: Read permission
    5: Read and execute permissions
    6: Read and write permissions
    7: Read, write, and execute permissions
