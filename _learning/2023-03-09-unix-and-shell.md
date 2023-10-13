---
title: "UNIX SHell"
permalink: /unix/
excerpt: "a place for my `sh` scripts and tricks I discover"
## last_modified_at: 2016-11-03T11:13:12-04:00
collection: learning

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

```shell
# thank me later
mkdir new_dir && cd $_

mkdir -p /not_yet_created_dir/new_dir && cd $_
```

## TODO / Master

- [ ] xargs
- [ ] parallel (gnu)
- [ ] grep
- [ ] find
- [ ] sed
- [ ] awk

```sh
# bulk file renaming
for f in ./*; do mv "$f" "${f/Silicon_Valley/S01E01}"; done
```

## `jq`

```sh
cat db/tickets.json | jq '.[].submitter_id' | wc -l
```

## `sed`

```sh
echo 's/hello/world/' > myscript.sed
sed -f myscript.sed input.txt > output.txt
```

### take the first line

```sh
# take the first line returned
some_command | sed -n '1p'

# or awk 
some_command | awk 'NR==1'

# or head
some_command | head -n 1
```

## `pbcopy` Apple and `xclip` Linux


```sh
cat file.txt | pbcopy
```

### combo with jq / head / copy
```sh
bat db/tickets.json | jq '.[]._id' -r | head -n 1 | xclip -selection clipboard
```

## xargs + find

```sh
find . -maxdepth 1 -name "*.md" | xargs tail
find . -name "*.txt" | xargs rm
ls | xargs rm
echo "dir1 dir2 dir3" | xargs mkdir
# Parallelizing commands:
find . -name "*.txt" | xargs -P 4 gzip
```

## Parallel

```sh
echo "file1 file2 file3" | parallel command-to-run {}
parallel echo "Hello, {}" ::: Alice Bob Charlie
ls *.txt | parallel wc -l
find *.md | xargs tail | parallel echo '------------------' | less

```

## Background jobs / Vi session

you can send current session into the background by `ctrl +z` (at least in `VI`)
use `fg` to list all jobs in the background, or switch into a job(tab after `fg`)

## Piping grep + awk + xargs

```sh
your_script | grep keyword_in_line | awk '{print $1}' | command_to_use_return_value_from_the_previous_step

## grab args
```sh
jobs -p | ag lala | awk '{print $3}' | xargs kill -9
## or
ps aux | grep ruby | awk '{ print $2 }'
```

```sh
echo foo$(you_script)bar
```

# Data Streams `stdin` `stdout` `stderr`

[Great Tutorial](https://www.youtube.com/watch?app=desktop&v=zMKacHGuIHI)

```sh
## stdin 0<
## stdout 1> # OR JUST stdout >
## stderr 2>
```

```sh
find /qwerty -type f 1> ./lala 2> ./err
## OR to APPEND instead of overwriting
find /qwerty -type f 1>> ./lala 2>> ./err
```

# Permissions

```sh
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

# Error debugging tools

1. `lsof`: "lsof" stands for "list open files." It displays information about files and network connections that processes have open on the system. It can help identify which processes are using specific files, sockets, or network connections.
2. `htop`: "htop" is an interactive process viewer that provides a real-time overview of system resource usage. It displays CPU and memory utilization, running processes, and allows for easy process management, such as killing processes or changing their priorities.
3. `ps`: "ps" is a command-line tool used to view information about running processes. It provides a snapshot of active processes, their resource usage, and process attributes like process IDs (PIDs), parent PIDs, CPU and memory usage, and more.
4. `strace`: "strace" is a powerful diagnostic tool that traces and records system calls made by a process and the signals received by it. It helps in analyzing the behavior of a program or troubleshooting issues related to system calls.
5. `tcpdump`: "tcpdump" is a network packet analyzer that captures and displays network traffic` in real-time. It can help debug network-related issues by capturing packets and allowing you to analyze their contents and flow.
6. `netstat`: "netstat" is a command-line tool used to display active network connections, routing tables, and various network interface statistics. It can provide information about open ports, established connections, and network-related statistics.
7. `dmesg`: "dmesg" displays the kernel ring buffer, which contains messages related to the system's hardware and software. It can be useful for troubleshooting hardware-related issues, device initialization problems, or kernel-level error messages.
8. `gdb`: "gdb" is a powerful debugger that allows you to analyze and debug programs. It enables you to inspect variables, set breakpoints, step through code, and track down the root cause of program crashes or abnormal behavior.


```sh
lsof -i -P | grep -i "listen"
# or
ps aux | grep postgres
```

brew services issues might be down and need / (re)starting
```sh
brew services list

brew services start postgres@12
```

sometimes temp file record  `pid` needs to be cleared manually

```sh
cat /opt/homebrew/var/postgres@12/postmaster.pid
rm /opt/homebrew/var/postgres@12/postmaster.pid
brew info postgresql@12
```

# Cron

```sh
crontab -l # list

crontab -e 
crontab -u root -e # for specific user, you can also set cron jobs for other users that you can't log into

@hourly echo 'hourly cron'
@reboot echo 'cron message triggered by reboot'

* * * * * echo "hello world"
* * 1 * * echo "happy new month!"
0 9 * * 1 echo "happy Monday"


0 9 * * 1 /usr/local/bin/script.sh # state full path of a command for other users
# save and exit! tail and see your echos

tail -f /var/log/syslog | grep echo

```

Cheatsheet / generator [https://crontab.guru](https://crontab.guru)

# Signals

```sh
kill -l # list signals

kill(3123, 15) # this is for sending signal 15 to a process 3123, but the syntax to shell is different in an opposite order
raise(signum) # to send a signal yourself

## Find a process that uses port and kill it

```sh
lsof -i :9292
kill -9 <PID>
# or
lsof -i :9292 | grep IP | awk '{print $2}' | xargs kill -9
```

# handy quirks

## cat

Append to file using `cat` and `HEREDOC`

```sh
cat <<EOF >> filename.txt
This is the first line to append.
This is the second line to append.
EOF
```

## rename

 rename
```sh
mv db/{oldName,newName}_structure.sql
```

## send session into the background / jobs
`In VIM ctrl + Z`	Backgrounds the job
`jobs`	To list all jobs
`fg` 	Foreground last job (fg 1 or 2 to select specific)


# Calendar

```sh
cal 2018
cal -3
```
