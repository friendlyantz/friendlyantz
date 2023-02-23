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
