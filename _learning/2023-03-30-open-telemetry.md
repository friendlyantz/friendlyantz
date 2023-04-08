---
excerpt: "My open telemetry notes"
collection: learning
categories:
  - software_engineering
  - learning
tags:
  - software_engineering
  - learning
  - open_telemetry
---

# Table Of Contens

* Table of contents (do not remove this line)
{:toc}

# My Ruby sandbox

[My Ruby sandbox](https://github.com/friendlyantz/open-telemetry-sandbox)

# Great Intro Video
[https://www.youtube.com/watch?v=Txe4ji4EDUA](https://www.youtube.com/watch?v=Txe4ji4EDUA)

# Theory 
## MELT - data types required for monitoring
[MELT 101](https://newrelic.com/platform/telemetry-data-101)
* Metrics - A numeric status at a moment in time (like CPU % used) OR Aggregated measurements (like a count of events over a one-minute time, or a rate of events-per-minute). Metrics require careful decision-making. Metrics work well for large bodies of data or data collected at regular intervals when you know what you want to ask ahead of time, but they are less granular than event data.
- Events - a history of every individual thing that happened in your system, you can roll them up into aggregates to answer more advanced questions on the fly. Events are great for answering questions about what happened in your system, but they are not as good for answering questions about what is happening in your system. Every event takes some amount of computational energy to collect and process. They also take up space in your database—potentially lots of space. So for relatively infrequent things, like a purchase in a vending machine, events are great, but we wouldn’t want to collect an event for everything the vending machine does.
* Logs - Similar to events, log data is discrete—it’s not aggregated—and can occur at irregular time intervals. Logs are also usually much more granular than events. In fact, one event can correlate to many log lines.
* Traces - or more precisely, “distributed traces”—are samples of causal chains of events (or transactions) between different components in a microservices ecosystem. And like events and logs, traces are discrete and irregular in occurrence. Traces that are stitched together form special events called “spans”; spans help you track a causal chain through a microservices ecosystem for a single transaction. To accomplish this, each service passes correlation identifiers, known as “trace context,” to each other; this trace context is used to add attributes on the span.

[metrics, tracing and logging article](https://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html)

## Important concepts of Context and Propagation

### 2 types of `Context`
- Span Context
  - Trace ID
  - Span ID
  - Trace Flags
  - Trace State

- Correlation Context
  - user-defined key-value pairs

### Propagation
Propagation is the mechanism that moves data between services and processes. Although not limited to tracing, it is what allows traces to build causal information about a system across services that are arbitrarily distributed across process and network boundaries.

---

# OTLP - wire protocol for OpenTelemetry (supports gRPC and HTTP over Protobu and HTTP over JSON)
<img width="722" alt="image" src="https://user-images.githubusercontent.com/70934030/228970631-a3c68289-5e16-4e54-acf3-00c5f1b0180e.png">
