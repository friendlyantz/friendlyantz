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
* Table of contents (do not remove this line)
{:toc}


### MELT - data types required for monitoring
[MELT 101](https://newrelic.com/platform/telemetry-data-101)
Metrics
Events
Logs
Tracing

[metrics, tracing and logging article](https://peter.bourgon.org/blog/2017/02/21/metrics-tracing-and-logging.html)
## My Ruby sandbox

[My Ruby sandbox](https://github.com/friendlyantz/open-telemetry-sandbox)

# Important concepts of Context and Propagation

## 2 types of `Context`
- Span Context
  - Trace ID
  - Span ID
  - Trace Flags
  - Trace State

- Correlation Context
  - user-defined key-value pairs

## Propagation
Propagation is the mechanism that moves data between services and processes. Although not limited to tracing, it is what allows traces to build causal information about a system across services that are arbitrarily distributed across process and network boundaries.
