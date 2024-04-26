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
3. `Threads`(aka `ServerThreds` or `ProcessorThreads`) - contain instances of `Sidekiq::Processors` that pull from queue or many queues and perform work. `Machines` have many `Processes`, and each `Process` has many `Threads`.

- `Sidekiq clients` Ruby objects that place jobs into queues. essentially  which can enqueue jobs, by adding keys to a Redis queue structure.
- `Sidekiq server` - A number of computers (that is, your servers or VPSs or Kubernetes nodes or whatever) contain a number of `Sidekiq processes` that do the work of pulling jobs from the queue and executing them.

![](https://www.mermaidchart.com/raw/21e7a3ee-4ee3-430a-8913-daba1f584084?theme=dark&version=v0.1&format=svg)

---

Sidekiq connects to Redis Pool, but never to Redis directly

---

## Sidekiq’s Redis commands

are generally executed serially, so we wait for a reply from the database before sending the next command. That means that 2 commands generally take 2 times as long to execute. It also means that we’re imposing 2 times as much load on the Redis database.

Another important thing when considering the Redis command required to queue or execute a job is time complexity (commonly notated with Big O notation). See also [Big O cheatsheet](https://www.bigocheatsheet.com/)

`perform_async` is fast `O(2)`, it uses:

- 2x Redis commands on insert(`SADD`) and ([LPUSH](https://redis.io/commands/lpush/)), and
- 1x on execution ([BRPOP](https://redis.io/commands/brpop/))

`perform_at` is SLOW 🐌 and is often used to "smear" load into the future:

```ruby
perform_at(Time. now + rand(100))
```

On insert: it calls [`ZADD`](https://redis.io/commands/zadd/) Redis command which adds to a sorted list and uses `O(log (N))`
On execution: moves jobs from the scheduled set to a queue with 3
commands:

```
ZRANGEBYSCORE   O(log(N)+M)
ZREM            O(M*log(N))
LPUSH           0(1)
```

In summary:
>
 perform_at turns
 3 fast commands into
 5 slow ones.

Redis is single-threaded and can only exec 1 command at a time

Job uniqueness checks also add lots of Redis back-and-forth(whether this is sidekiq pro/enterprise or 3rd party uniqness plugin)

Locks can be another source of extra Redis commands(locks from other plug ins or pro/entrprs) -> SUPER SLOW

Fan-outs(when a job schedules other jobs) create extra Redis load, but not that much

`push_bulk` use push bulk anywhere
possible - but the benefit is mostly in eliminating Round Trip Time (RTT)
Redis RTT matters here, too

# Ch2: Understanding Queueing Systems

Queueing theory provides a few bits of useful terminology:

1. Unit of Work. In Sidekiq, this is one Job request. In a grocery store, it might be one customer in line.
2. Server. In computers, we’re used to “server” meaning “one machine”. In queueing theory, “one server” is one “unit of parallel processing capacity”. If I have 1 “server”, I can process one unit of work at a time. If I have 2 servers, I can process 2 units of work at the same time (in parallel). I will also refer to this quantity as parallelism throughout Sidekiq in Practice. This is useful, because in some cases we can have “1.5 servers”, which just sounds weird.
3. Service Time. Service time is how long it actually takes to process a unit of work. This is how long a Sidekiq job takes to actually run.
4. Wait Time. How long units of work spend waiting in the queue.
5. Total Time. Service time plus wait time. This is the total time from a job being enqueued until it is finished.

## Little’s Law

Little’s Law is a very simple equation: it simply says that the number of “units of work” (e.g. customers in a line, jobs in our Sidekiq queues) in a queueing system is equal to the average amount of time required to process that work, multiplied by the average rate of arrival of work (how many customers arrive per minute, how many jobs arrive per minute).
The result of Little’s Law is also called offered traffic.

Queue Length Exponentially Increases as Utilization Increases

Generally, we can divide queueing systems into two regimes: low utilization and high utilization. When utilization of the system is between 0 and 70%, queue wait times increase quite slowly as utilization increases. But, when utilization increases beyond 70%, the exponential effects take over and queue latency increases quickly.

---

Wait Time Is Proportional to 1/Servers

> The “knee” can be made “deeper” by increasing the number of servers pulling from the queue. This is because wait time is inversely proportional to how many servers there are.

---

### A stable system when load rapidly increases

As load increases (jobs are enqueued more rapidly), the length of the queue will increase. Rapid growth of a queue can effectively cause a “brownout”, where the system is not technically down (it’s still online and processing jobs), but the queue has become so deep that by the time jobs execute, they may be completely irrelevant!

## USE Method

### Utilization

Utilization metrics are just ratios: resources used divided by resources available.
In a queueing system, the most important resources are not memory or CPU, but the servers which can do work.

### Saturation

Saturation is what occurs when our system is 100% utilized at any given moment: that is, saturation is the growth of the queue. In systems without queues, or with queues of limited length, saturation leads to rejection/denial of service.

Saturation metrics are all about queues. The most important one for Sidekiq will generally be the length of the queue in seconds, calculated by taking the next job in our queue and subtracting when it was enqueued from the current time.

### Errors

All errors are wasted capacity: wasted work that could have been spent doing something productive. Keeping errors low means we keep our server capacity available for useful work.
Useful error metrics to track for Sidekiq include:

- Size of the retry queue
- Size of the dead queue
- Redis connection errors
Error metrics should be as low as possible, so that capacity can be used for useful work.

The Ideal Sidekiq

1. Each job’s total time (time spent in the queue waiting plus time spent servicing the job in a Sidekiq thread) is less than or equal to its requirements, which vary based on the job.
2. Utilization is as high as possible while still meeting total time requirements.
3. Errors are low, so that the maximum amount of capacity is being used on useful, not wasteful, work.
4. The system can respond quickly to changes in load, keeping job “total time” within parameters even when lots of jobs arrive at once, without downtime.

# CH 3: Concurrency

**threads cost a lot of memory in CRuby**
When there is more than one thread in a process, multiple threads can be ready to run Ruby code, but only one can run it at a time. This creates queueing for the Global VM Lock, as threads wait for the GVL to free up. As we covered in the previous chapter, queueing increases latency.

**Setting higher than your database setting can slow down your Sidekiq by 100x.**

---
Video takeaways

- handfull of queues are better making SLO easier to read and monitor
- Floods of a particular job type can be solved with weighting + longer SLOs
- read-replica queues is a good stratagey. Consider Replica lagDB => read global txn_id to takle it (`perform(args, txn_id`) OR read replica lag(but this is not 100% accurate)

VID
---
