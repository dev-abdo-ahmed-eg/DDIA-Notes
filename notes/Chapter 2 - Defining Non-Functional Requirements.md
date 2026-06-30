# Chapter 2 - Defining Non-Functional Requirements

## What Are Non-Functional Requirements?

Non-functional requirements describe how a system should behave rather than what it should do.

Examples:

- Fast
- Reliable
- Secure
- Maintainable

A reliable service continues to work even when some parts of the system fail.

---

## Throughput

Throughput is the number of requests a system can process per second.

It is usually measured in:

- Requests per second (RPS)

Higher throughput means the system can handle more work in the same amount of time.

---

## Scalability

A system is considered scalable if its maximum throughput can be significantly increased by adding more computing resources.

Examples of computing resources:

- More CPU cores
- More memory
- More servers

The goal of scalability is to handle increasing load without a major drop in performance.

---

## Sources of Delays in a System

Many factors can add random delays to a request:

- Context switching to background processes
- Loss of network packets and TCP retransmissions
- Garbage collection pauses
- Page faults when reading data from disk
- Mechanical vibrations in storage hardware

Because of these factors, response times are not always consistent.

---

# Key Ideas

*   **Nonfunctional Requirements are Foundational:** While functional requirements define *what* an application does, nonfunctional requirements (reliability, scalability, maintainability) determine if it is actually usable. A system that is bearsably slow or unreliable might as well not exist.
*   **The Myth of the "Magic Scaling Sauce":** There is no single, one-size-fits-all architecture for scalability. An architecture that handles 100,000 requests per second looks fundamentally different from one handling 3 requests per minute, even if the total data throughput is identical.
*   **Maintenance as the Long-Term Cost:** The majority of a software system's cost is incurred during its ongoing maintenance (fixing bugs, adapting to new platforms, repaying technical debt) rather than its initial development.

---

# Core Concepts

*   **Reliability:** The ability of a system to continue working correctly even when things go wrong (hardware faults, software bugs, or human error).
*   **Fault vs. Failure:** A **fault** is a specific component deviating from its specification (e.g., one disk dying), whereas a **failure** is the entire system stopping the service for the user. Reliable systems are designed to be **fault-tolerant**, preventing faults from escalating into failures.
*   **Scalability:** A system's ability to cope with increased load (e.g., more users, higher data volume). It is not a binary "yes/no" label but a description of how a system responds to growth.
*   **Maintainability:** The ease with which a system can be operated and evolved by human teams over years. It consists of three pillars: **Operability** (ease for ops teams), **Simplicity** (managing complexity), and **Evolvability** (ease of making changes).
*   **Fan-out:** The factor by which one initial request results in several downstream requests being executed (e.g., one post reaching millions of followers).

**Core Performance Metrics**
* Response Time: The total time from when a user makes a request until they receive the answer
. This is a client-side view that includes all network delays and waiting time

* Throughput: The number of requests or the volume of data the system processes per second
. While users care about response time, architects use throughput to determine hardware requirements and costs

* Latency: Often used interchangeably with response time, but specifically refers to the time a request is latent (waiting) and not being actively processed, such as during network transit

![Architecture](<Screenshot 2026-06-23 211050.png>)

---

# Architecture & Design Trade-offs

*   **Vertical vs. Horizontal Scaling:** 
    *   **Vertical (Scaling Up):** Moving to a more powerful machine (more CPU/RAM); simpler to reason about but limited by hardware ceilings and cost.
    *   **Horizontal (Scaling Out):** Adding more machines (shared-nothing architecture); provides massive scale and fault tolerance but incurs the complexity of distributed systems.
*   **Read-time Joins vs. Write-time Materialization:**
    *   **On-demand Reads:** Simple write path, but read performance dies at scale when complex joins are required (e.g., fetching a home timeline in SQL).
    *   **Precomputed Materialization:** High-performance reads from a cache, but increases write load significantly due to fan-out and forces you to manage derived data consistency.
*   **Abstraction vs. Complexity:** Good abstractions (like SQL) hide implementation details to manage **accidental complexity**, but they can be leaky and hide the underlying performance trade-offs.

---

# Real-World Examples

*   **Social Network (Twitter) Timelines:** A hybrid approach handles scale: precomputing timelines for most users while treating "celebrity" accounts as a special case to avoid massive write fan-out.
*   **Amazon's p99.9 Response Time:** Amazon focuses on high percentiles because the users with the most data (and thus the slowest requests) are often their most valuable, high-purchasing customers.

---

# Common Misconceptions

*   **Averaging Response Times:** Using the arithmetic mean is misleading because it hides the **tail latency** experienced by the slowest requests.
*   **"X Scales, Y Doesn't":** Scalability should be discussed as a set of specific questions: "If we double the load, how much do we need to increase resources to keep performance constant?".

---

# Discussion Points

*   **Why is p99 latency often more important than the average?** To identify the experience of your most affected (and potentially most valuable) users and because slow requests at the tail often point to system bottlenecks like queueing delays.
*   **How do you handle the "celebrity problem" in a write-heavy fan-out system?** By using a hybrid architecture: precompute for standard users, but merge celebrity posts at read-time to prevent write-side overload.
*   **Explain the difference between Accidental and Essential complexity.** Essential complexity is inherent in the problem; accidental complexity is introduced by poor tooling or bad design (which abstractions aim to fix).

# What To Remember

*   **Plan for 10x, not 100x:** Build an architecture that can handle an order of magnitude more load, but don't over-engineer for 100x growth, as you will likely need to rethink the architecture entirely by that point anyway.
*   **Simplicity is a Maintainability Goal:** A "big ball of mud" system with hidden assumptions and tight coupling is a liability that grows more expensive every year.
*   **Distribution is a Choice, Not a Default:** If a single-machine database can handle your load, it is almost always simpler and cheaper than a distributed setup.!

---

## My Takeaways

One of the most useful ideas I got from this chapter is that when a user sees a timeout, it does not automatically mean that the server is down.

In a typical system:

```text
   Tablet App 
        ↓
       IIS
        ↓
    Web API
        ↓
      EF6
        ↓
    SQL Server
```

a request can fail or timeout at multiple layers.

Possible timeout scenarios include:

### Client Timeout

The client stops waiting before the server finishes processing.

The request may still be running on the server, and the operation may even complete successfully.

### IIS / ASP.NET Timeout

The request exceeds the configured execution time and ASP.NET terminates it before completion.

### Thread Pool Starvation

The application is running, but all available worker threads are busy.

New requests wait in line and may eventually timeout.

### Request Queue Overflow

Too many incoming requests fill the ASP.NET request queue.

Some requests may be rejected before reaching application code.

### Command Timeout (ADO.NET / EF6)

A slow database operation exceeds the configured `CommandTimeout`.

An important observation is that the timeout is usually triggered by the client library (ADO.NET), not by SQL Server itself.

### SQL Blocking

A query may spend most of its time waiting for locks held by another transaction.

The application sees a timeout, but the root cause is blocking rather than query execution itself.

### Connection Pool Exhaustion

All database connections are in use.

New requests wait for an available connection and may eventually fail with a timeout.

### Network Issues

The request may fail because of network connectivity problems between the client and the server.

## Key Lesson

When investigating a timeout, the most important question is not:

> "Which component reported the timeout?"

but rather:

> "Which component caused the delay?"

The component that reports the timeout is often different from the component that introduced the latency.

This distinction is essential when diagnosing distributed systems and production incidents.
