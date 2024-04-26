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
3. `Threads`(aka `ServerThreds` or `ProcessorThreads`) - contain instances of `Sidekiq::Processors` that pull from queue or many queues and perform work. `Machines` have many `Processes`, and each `Process` has many `Threads`. Processors call `perform` method on workers

- `Sidekiq clients` Ruby objects that place jobs into queues. essentially  which can enqueue jobs, by adding keys to a Redis queue structure.
- `Sidekiq server` - A number of computers (that is, your servers or VPSs or Kubernetes nodes or whatever) contain a number of `Sidekiq processes` that do the work of pulling jobs from the queue and executing them.

![](https://www.mermaidchart.com/raw/21e7a3ee-4ee3-430a-8913-daba1f584084?theme=dark&version=v0.1&format=svg)

---

Sidekiq connects to Redis Pool, but never to Redis directly

---

1. Job/Worker:
	1. SomeJob.set(queue: 'foo').perform_async(....)
	2.  client_push(item)
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

`perform_at` is 🐌 `O(log (N))`and is often used to "smear" load into the future:
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
