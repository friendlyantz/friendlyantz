---
title: Queueing theory and Sidekiq practice
excerpt: "background job learnings and notes on Book: 'Sidekiq in Practice'"
permalink: /sidekiq
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - book
  - background_jobs
toc: true
toc_sticky: true
---

# Ch 1: How Sidekiq works

Main Sidekiq components: job/work, queues, and threads

1. `Job` alias for `Worker` - a request to do work = tuple [JobClass, args],  Ruby classes have the `perform method` defined. They represent the work we want done.

```ruby
# sidekiq/worker.rb
def perform_async(*args)
  client_push("class" => self, "args" => args)
end
```

2. `Queues` are list data structures in Redis that hold tuples of [job_class, arguments]. They contain individual requests for running a job.
3. `Threads`(aka `ServerThreds` or `ProcessorThreads`) - contain instances of `Sidekiq::Processors` that pull from queue or many queues and perform work. `Machines` have many `Processes`, and each `Process` has many `Threads` which depend on concurrency. `INCREASES CONCURRENCY`
 - Processors call `perform` method on workers.
    i.e. of a process on my laptop (`server`) can be invoked via `sidekiq -r some_job.rb`, which will have `5 threads` (or whatever we give it). I can also run another process in another terminal windows with the same command
 1. (# ~~The Processor is a standalone thread~~ - TBC, most likely taken from vid)

- `Sidekiq clients` Ruby objects that place jobs into queues. essentially  which can `enqueue` jobs, by adding keys to a Redis queue structure. Some enqueue, that can `Sidekiq::Client.push_bulk("class" => Smth, "args" => [1,2,3])`, or I can just run in `irb` something like `SomeJob.perform_async(args)` TBC
- `Sidekiq server` - A number of computers (that is, your servers or VPSs or Kubernetes nodes or whatever) contain a number of `Sidekiq processes` that do the work of pulling jobs from the queue and executing them. `INCREASES PARALLELISM`

![](https://www.mermaidchart.com/raw/21e7a3ee-4ee3-430a-8913-daba1f584084?theme=dark&version=v0.1&format=svg)

---

Sidekiq connects to Redis Pool, but never to Redis directly

---

1. Job/Worker:
 1. SomeJob.set(queue: 'foo').perform_async(....)
 2. client_push(item)
2. Client
 1. push(item) --> raw_push(payloads) to @redis_pool
  1. multi connection atomic push

## Sidekiq’s Redis commands

- commands executed serially.
- `Redis` is single-threaded and can only exec 1 command at a time
 	- for Redis scaling - we want a lot of small commands, instead of fewer long slow commands. don't lock the loop!!!
- ✅`perform_async` is better than `perform_at`
 	- consider BigO of each command. See also [Big O cheatsheet](https://www.bigocheatsheet.com/)
- 🛑`Job Uniqueness` libraries kill performance, avoid them. Job's should not rely on idempotency and should be written correctly
- 🛑`Locks` can be another source of extra Redis commands(locks from other plug ins or pro/enterprise) -> SUPER SLOW.
- 🚧`Fan-outs`(when a job schedules other jobs) create extra Redis load, but not that much
- ✅`push_bulk` use it anywhere possible! it eliminates `Round Trip Time (RTT)` and waiting for connection
- 🚧`Thread` count (therefore Redis connection count) slows down Redis:
 	- i.e. 100-connections DB can support twice ops/sec more that 3000-connections DB

---

`perform_async` is fast `BigO(2)` ⚡️

- On insert: ([SADD](https://redis.io/docs/latest/commands/sadd/)) and ([LPUSH](https://redis.io/commands/lpush/)), and
- On execution: ([BRPOP](https://redis.io/commands/brpop/))

```
SADD            0(1)
LPUSH           0(1)

BRPOP           0(N)
```

`perform_at` is 🐌 `O(n log (n))`and is often used to "smear" load into the future:

- On insert: it calls [`ZADD`](https://redis.io/commands/zadd/) Redis command which adds to a sorted list
- On execution: moves jobs from the scheduled set to a queue with 3
commands:

```
ZADD            O(log(N))           scheduling (instead of BigO(1) for SADD)

ZRANGEBYSCORE   O(log(N)+M)         runs serally(not pipelined) with ZREM
ZREM            O(M*log(N))         runs serallynot pipelined) with ZRANGEBYSCORE
LPUSH           0(1)
```

> `perform_at` turns 3 fast commands into 5 slow ones.

---

# Ch2: Understanding Queueing Systems

Queueing theory provides a few bits of useful terminology:

1. Unit of Work - job
2. Server - one “unit of parallel processing capacity”.
3. Service Time - actual job execution time
4. Wait Time
5. Total Time

## Little’s Law

Little’s Law:

```ruby
“units of work” (aka OFFERED TRAFFIC) ==  
 "the average amount of time required to process that work / service time" 
 * "the average rate of arrival of work (how many jobs arrive per sec)."
# this relates directly to how many of our underlying “servers” (in queueing theory) we need
# OFFERED TRAFFIC = 10sec / (120 jobs / 60 sec) => 5 jobs offered


utilization == OFFERED TRAFFIC / parallelism
# utilization 50%: 5 (offered traffic) / 10 (parallelism). If you had 10 single-threaded Sidekiq processes pulling from this queue

# on a truly parallel JRuby or TruffleRuby, we would count the number of threads,
# on MRI Ruby however, the amount of parallelism offered by a multi-threaded Sidekiq process can be complicated to think about,
# mathematical equation called Amdahl’s Law is used instead
```

---

Queue Length Exponentially Increases as Utilization Increases

Generally, we can divide queueing systems into two regimes:

1. low utilisation
2. high utilisation.
 
```ruby
case utilization
when 0..70%
 queue wait times increase quite slowly as utilization increases
when 70..
 the exponential effects take over and queue latency increases quickly.
end
```

---

⚡️Wait Time Is Proportional to 1/Servers

> The “knee” can be made “deeper” by increasing the number of servers pulling from the queue. This is because wait time is inversely proportional to how many servers there are.

> Queues which have more servers pulling from them are generally better for utilization and cost efficiency
---

### A stable system when load rapidly increases

As load increases (jobs are enqueued more rapidly), the length of the queue will increase. Rapid growth of a queue can effectively cause a “brownout”, where the system is not technically down (it’s still online and processing jobs), but the queue has become so deep that by the time jobs execute, they may be completely irrelevant!

## USE Method (Utilization - Saturation - Errors)

>by Netflix engineer Brendan Gregg
>
### Utilization

```ruby
resources_used / resources_available

Sidekiq::WorkSet.new.size  / Sidekiq::ProcessSet.new.total_concurrency
# the most important resources are not memory or CPU,
# but the servers which can do work.
```

- instantaneous
- sampled (i.e. over 15mins or 1 minute, etc.)

### Saturation

Saturation metrics are all about queues. The most important one for Sidekiq will generally be the length of the queue in seconds

```ruby
the_length_of_the_queue_in_seconds =
 next_job_in_queue - when_it_was_enqueued_from_the_current_time
 
Sidekiq::Queue.new('queue_name').latency.round(2)
# If the next job was enqueued 5 minutes ago, our queue length is 5 minutes.
```

Saturation is what occurs when our system is 100% utilized at any given moment. In systems without queues, or with queues of limited length, saturation leads to rejection/denial of service.

Control saturation by decreasing utilization:

1. the most common is adding more Sidekiq `server` processes,
2. but we can also sometimes increase Sidekiq’s `concurrency` setting to gain more `parallelism`

### Errors

- Size of the retry queue
- Size of the dead queue
- Redis connection errors

## The Ideal Sidekiq

1. Each job’s `total time` is less than or equal to its requirements, which vary based on the job.
2. `Utilization` is as high as possible while still meeting total time requirements.
3. `Errors` are low, so that the maximum amount of capacity is being used on useful, not wasteful, work.
4. The system `can respond quickly` to changes in load, keeping job “total time” within parameters even when lots of jobs arrive at once, without downtime.

----

## Lab takeaways

`Sidekiq::Stats.new` are handy, but wont get your as far as digging through `Sidekiq::CLI.instance`

```ruby
# UTILIZATION
busy_threads_count = Sidekiq::WorkSet.new.size # number of BUSY threads on server
busy_threads_count = Sidekiq::Stats.new.workers_size # same as aboeve

all_threads_count = Sidekiq::ProcessSet.new.total_concurrency # number of NOT BUSY and BUYS threads on server
util = busy_threads_count.to_f / all_threads_count * 100


# Alternative, but for one of the processes on server
worker_set = Sidekiq::CLI.instance&.launcher&.managers&.first&.workers
    busy_threads_count_for_process = worker_set&.count(&:job) # only busy threads in this process
    all_threads_count_for_process = worker_set&.count # all threads in this process, it depends on concurrency
util = busy_threads_count_for_process.to_f / all_threads_count_for_process * 100
```

```ruby
# SATURATION
# current queue depths
Sidekiq::Stats.new.default_queue_latency.round(2)
# or
Sidekiq::Queue.new('retry').latency.round(2)

Sidekiq::Stats.new.retry_size 
```

for custom concurrency

```ruby
be sidekiq -c 2-r ./app.rb # will do only 2 threads
```

----

# CH 3: Concurrency

> fewer threads is often better. default was changed from 25 to 10 in Sidekiq 5.2.0.

>**threads cost a lot of memory in CRuby**, often caused by `memory fragmentation`

When there is more than one thread in a process, multiple threads can be ready to run Ruby code, but only one can run it at a time. This creates queueing for the Global VM Lock, as threads wait for the GVL to free up.

**Setting higher than your database setting can slow down your Sidekiq by 100x.**

1 thread uses `log(x)` memory over time (where time is x),
so increasing the number of threads leads to something like `n * log(x)` memory use.

---

- concurrency is when the start and end times of 2 or more operations overlap,
- parallelism is when we’re actually working on those operations at the same instant.

> All parallel operations are concurrent, but not all concurrent operations are parallel.

## exceptions

If you’re doing an extremely high amount of I/O in your Sidekiq jobs (90-95% I/O), high thread settings may still be useful to you. If all of your Sidekiq jobs spend 950 milliseconds waiting on a network call from Mailchimp and 50 milliseconds running Ruby code, thread counts as high as 128 may be useful to you. This information is generally available in any APM tool: New Relic, Datadog, etc. Memory fragmentation costs will be extremely high at these high concurrency settings, but your threads would probably still be much more efficient

---

| I/O Wait <br>(i.e API call, DB access <br>- use APM to find it out) | concurrency | parallelism (approx) |
| ------------------------------------------------------------------- | ----------- | -------------------- |
| 5% or less                                                          | 1           | 1                    |
| 25%                                                                 | 5           | 1.25                 |
| 50%                                                                 | 10 default  | 2                    |
| 75%                                                                 | 16          | 3                    |
| 90%                                                                 | 32          | 8                    |
| 95%                                                                 | 64          | 16                   |
most applications fall in the 50-75%-of-a-jobs-time-is-in-IO range, so the default concurrency setting of 10 is great for most

---
Video takeaways

- handfull of queues (i.e. `under minute`, `under_hour`, `under_6_hours` are better making SLO easier to read and monitor
- Floods of a particular job type can be solved with weighting + longer SLOs
- read-replica queues is a good stratagey. Consider Replica lagDB => read global txn_id to takle it (`perform(args, txn_id`) OR read replica lag(but this is not 100% accurate)

---

# Ch4 Parallelism. Locations of Saturation

- `Sidekiq processes`, because each has its own GVL, are always working in `parallel`.
- but `Threads` in CRuby are only working in parallel part of the time.

- In JRuby or TruffleRuby, threads can work (almost) entirely in parallel. So there thread == a queueing theory “server”
- in MRI / CRubyeach `Ractor` is a queueing theory “server”, because `Ractors` can execute in parallel.

- If you’re using CRuby, only run 1 Sidekiq process per vCPU/CPU core available to the machine.
- With JRuby or TruffleRuby, you’ll want one Sidekiq thread per core, so concurrency should not exceed the core count.

## Scaling additional

you should reserve enough parallelism to cover the minimum of your offered traffic, and then use spot instances to cover the difference between the minimum and maximum of your offered traffic requirements.

## Locations of Saturation

### 1. Queueing for system resources ⚡️

- ⚡️ If you’re using CRuby, only run 1 Sidekiq process per vCPU/CPU core available to the machine
 	- If you’re running out of memory, either get bigger machines with a higher GB-of-memory-to- vCPU ratio, or reduce (but be aware that the latter option reduces parallelism, as above
- With JRuby or TruffleRuby, you’ll want one Sidekiq thread per core

### 2. Queueing for Redis

the amount of transactions that a Redis database can handle per-second is proportional to the size of the keys.
 This means that a Sidekiq system with smaller arguments will scale better, because small arguments mean small Redis keys, leading to more Redis operations per second.

### 3. Queueing for your Main Database

1. Connection pool exhaustion. (i.e. pgbouncer - outgoing connections get saturated)
2. ActiveRecord pool exhaustion. (see database.yml)
3. Resource exhaustion at the DB server. (less common than above)

### 4. Queueing for External APIs

# Ch5

## Rate limiting API calls

use Sidekiq Enterprise or
manually -> i.e. sleep 1 sec, if job is 0.2sec and concurrency is 5

```ruby
class ExternalAPIJob  
 include Sidekiq::Worker  
 MINIMUM_DURATION = 1 # Seconds. Tuned for concurrency 5.

 def perform(id)  
  time = Time.now  
  # send request here  
  if Time.now < time + 1
   sleep(time + 1 - Time.now)
  end
 end 
end
```

# Ch6

 things can prevent us from using 100% of our CPU:

- memory limits
- i/o wait var
- DB pooling
 our goal will be to meet our job latency requirements with the least expensive hardware possible.
