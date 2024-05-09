---
# layout: posts
# author_profile: true
# title: "Vi Everywhere"
# permalink: /life/pages/
excerpt: "WIP"
# last_modified_at: 2016-11-03T11:13:12-04:00
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - template
toc: true
toc_sticky: true
# toc_label: "My Table of Contents"
# toc_icon: "cog"
# header:
#   video:
#     id: _mrF0pp8LIE
#     provider: youtube

---



# Background jobs


---

- Jobs (old name _Workers_ )
- Queues
- Threads
- Servers --> Processes --> Threads
- Client

---

## What queues to have

- asap
- under 1min
- under 1 hour
- under 1 day

---

## how to schedule

```ruby
`perform_async`/`perform_later` 
vs 
`perform_at`/`perform_now`

```

`BigO(N)` 
vs 
`BigO(N * log (N)`

---

`perform_async`
```
SADD            0(1)
LPUSH           0(1)

BRPOP           0(N)
```

---

`perform_at`
turns 3 fast commands 
into 5 slow ones.

```
ZADD            O(log(N))           scheduling (instead of BigO(1) for SADD)

ZRANGEBYSCORE   O(log(N)+M)         runs serially(not pipelined) with ZREM
ZREM            O(M*log(N))         runs serially(not pipelined)
LPUSH           0(1)
```

---
# what metrics?

---

## 'USE' Method 

Utilization - Saturation - Errors

---

### Utilization

```ruby
resources_used / resources_available

Sidekiq::WorkSet.new.size  /  # busy thread count 
Sidekiq::ProcessSet.new.total_concurrency # total thread count

# the most important resources are not memory or CPU,
# but the servers which can do work.
```

- instantaneous 
- sampled (i.e. over 15mins or 1 minute, etc.)

---

### Saturation


```ruby	
Sidekiq::Queue.new('queue_name').latency.round(2)
# If the next job was enqueued 5 minutes ago, our queue length is 5 minutes.
```

---


Control saturation by decreasing utilization:
1. adding more Sidekiq `server processes`,
2. increase  `concurrency` setting to gain more `parallelism`

---

### Errors

- Size of the retry queue
- Size of the dead queue
- Redis connection errors

---

## The Ideal Setup

1. Each job’s `total time` is less than or equal to its requirements, which vary based on the job.
2. `Utilization` is as high as possible while still meeting total time requirements.
3. `Errors` are low, so that the maximum amount of capacity is being used on useful, not wasteful, work.
4. The system `can respond quickly` to changes in load, keeping job “total time” within parameters even when lots of jobs arrive at once, without downtime.

---

# Concurrency

| I/O Wait <br>(i.e API call, DB access <br>- use APM to find it out) | concurrency | parallelism (approx) |
| ------------------------------------------------------------------- | ----------- | -------------------- |
| 5% or less                                                          | 1           | 1                    |
| 25%                                                                 | 5           | 1.25                 |
| 50%                                                                 | 10 default  | 2                    |
| 75%                                                                 | 16          | 3                    |
| 90%                                                                 | 32          | 8                    |
| 95%                                                                 | 64          | 16                   |


---

- `Sidekiq Processes` are always working in `parallel` (own GVL)
- `Sidekiq Threads` in CRuby are only working in parallel part of the time.

For CRuby, only run 1 Sidekiq process per vCPU/CPU core available to the machine 

---
## redis optimisation

smaller args - better, Sidekiq will scale better, due to increased Redis ops/sec

the amount of transactions that a Redis database can handle per-second is proportional to the size of the keys. 


---

---


