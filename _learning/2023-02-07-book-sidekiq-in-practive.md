---
title: "Book: 'Sidekiq in Practice'"
excerpt: "Queueing system for Ruby"
collection: learning
---

These three components - work, queues, and threads

- A number of computers (that is, your servers or VPSs or Kubernetes nodes or whatever) contain a number of Sidekiq _processes_, referred to in Sidekiq as “server” processes. Sidekiq processes “do the work”, as opposed to...
- Sidekiq _clients_ place jobs into queues. Sidekiq clients are simply Ruby objects which can enqueue jobs. They are usually in a seperate process from the Sidekiq server process, but Sidekiq clients can be instantiated anywhere (jobs can enqueue other jobs, for example). They place jobs into queues by adding keys to a Redis queue structure.
- _Jobs_ are Ruby classes which have the perform method defined and include Sidekiq::Worker. They represent the work we want done.
- _Queues_ are data structures in Redis that hold tuples of [job_class, arguments]. They contain individual requests for running a job.
- Each Sidekiq process has a number of _Processor threads_. So, many machines “have many” processes, which have many threads. These threads contain instances of the Sidekiq::Processor class, so we call them Processor threads or just server threads. The threads in a Sidekiq server pull jobs from one or more queues and perform the work. They pull they oldest job from the queues they are assigned to process; Sidekiq is “first in, first out” (FIFO).

---

Sidekiq’s Redis commands are generally executed serially, so we wait for a reply from the database before sending the next command. That means that 2 commands generally take 2 times as long to execute. It also means that we’re imposing 2 times as much load on the Redis database.

Another important thing when considering the Redis command required to queue or execute a job is time complexity (commonly notated with Big O notation). 

---

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

---

A stable system when load rapidly increases
As load increases (jobs are enqueued more rapidly), the length of the queue will increase. Rapid growth of a queue can effectively cause a “brownout”, where the system is not technically down (it’s still online and processing jobs), but the queue has become so deep that by the time jobs execute, they may be completely irrelevant!

### USE Method
Utilization
Utilization metrics are just ratios: resources used divided by resources available.
In a queueing system, the most important resources are not memory or CPU, but the servers which can do work.

Saturation
Saturation is what occurs when our system is 100% utilized at any given moment: that is, saturation is the growth of the queue. In systems without queues, or with queues of limited length, saturation leads to rejection/denial of service.

Errors
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

