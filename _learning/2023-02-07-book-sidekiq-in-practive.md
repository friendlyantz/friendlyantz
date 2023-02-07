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

