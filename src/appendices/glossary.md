# Glossary

- [Solutions Architect](#solutions-architect)
- [Streaming](#streaming)
- [Unbounded data](#unbounded-data)
- [Unbounded data processing](#unbounded-data-processing)
- [Low-latency, approximate, and/or speculative results](#low-latency-approximate-andor-speculative-results)
- [Lambda Architecture](#lambda-architecture)

## Solutions Architect

A **solutions architect** is an IT professional who designs comprehensive technical solutions to meet specific business needs and requirements. They serve as a bridge between business objectives and technical implementation.

Key aspects of the role include:

**Core responsibilities:**
- Analyzing business problems and translating them into technical requirements
- Designing end-to-end system architectures that integrate multiple technologies, platforms, and services
- Making strategic decisions about technology choices, frameworks, and infrastructure
- Ensuring solutions are scalable, secure, reliable, and cost-effective

**Skills and focus areas:**
- Deep understanding of various technologies (cloud platforms, databases, APIs, networking, security)
- Ability to see the "big picture" while understanding technical details
- Strong communication skills to explain complex technical concepts to non-technical stakeholders
- Knowledge of software development, infrastructure, and enterprise systems

**Typical work:**
A solutions architect might design how a company's e-commerce platform integrates with inventory systems, payment processors, and customer databases, choosing appropriate cloud services, security measures, and scalability approaches to meet both current needs and future growth.

The role is particularly common in cloud computing (AWS Solutions Architect, Azure Solutions Architect) and enterprise software implementations, where they ensure all components work together effectively as a complete solution rather than just individual parts.

## Streaming

> Tyler Akidau's definition: *A type of data processing engine that is designed with infinite data sets in mind.*

## Unbounded data

> Tyler Akidau's definition: *A type of ever-growing, essentially infinite data set.*

**Unbounded data** refers to data that has no defined end point and continues to arrive indefinitely over time. This is also commonly called "streaming data" or "continuous data."

Key characteristics:

- **No fixed size or duration** - Unlike bounded data (like a static database or a CSV file), unbounded data keeps growing as new events occur
- **Time-based** - Data arrives continuously, often in real-time or near real-time
- **Processing challenges** - You can't wait for "all the data" before processing, since it never stops coming

**Common examples:**
- Sensor readings from IoT devices
- Website clickstreams and user activity logs
- Financial market data feeds
- Social media posts
- Server logs and application metrics

In data processing frameworks (like Apache Flink, Apache Beam, or Kafka Streams), unbounded data requires different processing approaches than traditional batch processing. Instead of processing a complete dataset at once, you typically use windowing strategies to group data into manageable chunks based on time or event counts.

## Unbounded data processing

> Tyler Akidau's definition: *An ongoing mode of data processing, applied to the aforementioned type of unbounded data.*

**Unbounded data processing** is a mode of data processing designed to handle continuously generated, infinite datasets that have no defined beginning or end point.

### Key Characteristics:

**Data nature**: The data being processed is unbounded—it arrives continuously over time (like user activity streams, IoT sensor readings, or financial transactions) rather than being a fixed, complete dataset.

**Processing approach**: The system processes data in an ongoing manner, continuously consuming and analyzing new data as it arrives, rather than operating on a static snapshot.

**Execution independence**: Importantly, unbounded data processing is **not synonymous with "streaming"** (as Akidau emphasizes). It describes the *nature of the data and processing mode*, not the execution engine. You can process unbounded data using:
- Streaming engines (like Apache Flink or Kafka Streams)
- Repeated batch jobs (running periodically on windows of unbounded data)
- Micro-batch systems (like Spark Streaming)

### Why the Term Matters:

The distinction exists to avoid confusion between:
- **What you're processing** (unbounded vs. bounded data)
- **How you're processing it** (streaming engine vs. batch engine)

A batch engine running every hour on new data is still doing unbounded data processing, even though it's not using streaming execution. Similarly, a streaming engine can process bounded datasets (like a fixed historical dataset) even though it's designed for continuous operation.

This terminology helps separate the conceptual problem (dealing with infinite data) from the implementation choice (which execution engine to use).

## Low-latency, approximate, and/or speculative results

This refers to a system design approach that prioritizes **speed of response** over perfect accuracy or completeness. It's commonly used in distributed systems, search engines, and real-time applications.

### Key components:

**Low-latency**: The system returns results very quickly, minimizing wait time. This often means setting strict time limits on operations rather than waiting for complete results.

**Approximate**: Results are "good enough" rather than exact. For example, a search engine might return the top 10 results from a subset of its index rather than scanning every document to find the absolute best matches.

**Speculative**: The system makes educated guesses or executes multiple approaches in parallel, using whichever completes first. For instance, it might query multiple data sources simultaneously and return the fastest response, or predict what a user will need before they explicitly ask for it.

### Common trade-off:

This approach exchanges some degree of accuracy, completeness, or certainty for significantly improved responsiveness. It's particularly valuable in scenarios where a fast, reasonably good answer is more useful than a perfect answer that arrives too late—like autocomplete suggestions, real-time analytics dashboards, or recommendation systems.

The classic example is Google search: it doesn't examine every page on the internet for your query, but quickly returns highly relevant results from a strategic subset, delivering them in milliseconds rather than minutes.

## Lambda Architecture

Lambda Architecture is a data processing design pattern that runs two parallel systems to handle large-scale data:

1. **A streaming layer** (called the "speed layer") that processes data in real-time, providing fast but potentially imperfect results
2. **A batch processing layer** that periodically reprocesses the same data to generate accurate, authoritative results

The architecture then merges outputs from both layers to give users low-latency access to data while ensuring eventual correctness.

**Why it was created:** Proposed by Nathan Marz, Lambda Architecture emerged as a pragmatic solution during an era when streaming systems couldn't guarantee correctness and batch systems were too slow for real-time needs. It allowed organizations to get the best of both worlds—immediate insights from streaming plus reliable results from batch processing.

**The tradeoff:** While effective, Lambda Architecture requires building and maintaining two separate pipelines that perform essentially the same computation, which creates significant operational complexity. You're essentially solving the same problem twice with different tools, then reconciling the results.

The pattern represented an important stepping stone in data architecture evolution, though modern streaming platforms with improved correctness guarantees have reduced the need for this dual-pipeline approach in many use cases.

## Exactly-once processing
**Exactly-once processing** is a guarantee in distributed systems and stream processing that ensures each message or event is processed one time and one time only—no more, no less—even in the presence of failures.

This means:
- **No duplicates**: If a failure occurs after processing but before acknowledging completion, the system won't reprocess the same message
- **No losses**: Messages aren't skipped or lost during processing
- **Idempotent outcome**: The end result is as if each message was processed exactly once, regardless of retries or failures

This is the strongest delivery guarantee in distributed systems, contrasting with:
- **At-most-once**: Messages may be lost but never duplicated (fast but unreliable)
- **At-least-once**: Messages are never lost but may be processed multiple times (reliable but requires handling duplicates)

Exactly-once processing is particularly important in scenarios like financial transactions, inventory management, or analytics where duplicate processing could cause serious problems (like charging a customer twice or double-counting metrics).

In practice, true exactly-once processing is difficult to achieve across all system boundaries. Many systems implement "effectively-once" semantics using techniques like idempotent operations, deduplication, and transactional guarantees to achieve the same practical outcome.

## Varying event-time skew
"Varying event-time skew" refers to a situation in stream processing or event-driven systems where the difference between when events actually occur (event time) and when they are processed or observed by the system (processing time) is inconsistent and changes over time.

In more detail:

- **Event time** is the timestamp indicating when an event actually happened in the real world
- **Processing time** is when the system receives and processes that event
- **Skew** is the gap between these two timestamps

The "varying" aspect means this gap is not constant—it fluctuates due to factors like network delays, system load, out-of-order event arrival, or temporary outages. For example, some events might arrive with only a 1-second delay, while others from the same stream might arrive minutes or hours late.

This concept is particularly important in systems that need to handle real-time analytics, windowing operations, or time-based aggregations, because the varying skew makes it challenging to determine when all events for a given time window have arrived and when it's safe to compute final results.

## Windowing
In the context of data processing, **windowing** refers to the technique of dividing a continuous stream or large dataset into smaller, manageable chunks called "windows" for processing.

### Key aspects:

**Purpose**: Windowing allows you to perform operations on subsets of data rather than waiting for an entire dataset or handling infinite streams all at once.

**Common types**:
- **Tumbling windows**: Fixed-size, non-overlapping segments (e.g., every 5 minutes)
- **Sliding windows**: Fixed-size segments that overlap (e.g., 5-minute windows sliding every 1 minute)
- **Session windows**: Dynamic segments based on periods of activity separated by gaps of inactivity
- **Global windows**: The entire dataset treated as one window

**Typical uses**:
- Stream processing (analyzing real-time data like sensor readings, logs, or user events)
- Time-series analysis (aggregating metrics over time intervals)
- Signal processing (analyzing audio or video in segments)
- Batch processing (breaking large datasets into processable chunks)

**Example**: If you're analyzing website traffic, you might use 1-hour tumbling windows to count page views per hour, or 10-minute sliding windows (updated every minute) to detect sudden traffic spikes more responsively.

Windowing is fundamental in frameworks like Apache Flink, Apache Spark Streaming, and Apache Beam, where it enables efficient aggregations, joins, and analytics on streaming data.

## Server Sent Events

Server-Sent Events 10 (SSE) is an option you can leverage to implement HTTP
streaming. SSE is a server push technology commonly used to send message updates or
continuous data streams to a browser client. SSE aims to enhance native, cross-browser
server-to-client streaming through a JavaScript API called EventSource, standardized 11 as
part of HTML5 by the World Wide Web Consortium (W3C)