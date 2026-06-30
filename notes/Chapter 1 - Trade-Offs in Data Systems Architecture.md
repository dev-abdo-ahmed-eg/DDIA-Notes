# Chapter 1 — Trade-Offs in Data Systems Architecture

## Topics

* Operational vs Analytical Systems
* Cloud vs Self-Hosting
* Distributed vs Single-Node Systems

---

## Application Types

### Data-Intensive

Data management is the primary challenge.

### Compute-Intensive

Computation is the primary challenge.

---

## Operational Systems

* Serve end-user requests.
* Read/write operational data.
* Optimize for low-latency transactions.

### Examples

* E-commerce
* Banking
* User Management

---

## Analytical Systems

* Serve business analysis workloads.
* Operate on a copy of operational data.
* Optimize for complex queries and aggregations.

### Examples

* Reporting
* Dashboards
* Business Intelligence

---

## Key Distinction

| Operational             | Analytical           |
| ----------------------- | -------------------- |
| User-facing             | Business-facing      |
| Transactional workloads | Analytical workloads |
| Frequent reads/writes   | Mostly read-only     |
| Low latency             | Large-scale queries  |


---

# Key Ideas

*   **System Design as Composition:** Modern applications are rarely built with one tool.
*   They are a **composition of specialized building blocks**
*   (databases, caches, search indexes, stream processors) integrated through application code,.
*   **The "No One Size Fits All" Rule:** No single database or storage engine is perfect for every task.
*   **Specialization via Scale:** As workloads grow, systems move away from general-purpose tools toward specialized architectures (e.g., separating transactional from analytical workloads) to manage performance and data volume.

# Core Concepts

*   **Data-Intensive:** Applications where the primary challenges are the volume, complexity, or rate of change of data, rather than raw CPU-bound computation.
*   **OLTP (Online Transaction Processing):** Operational systems that handle a high volume of small, low-latency queries and updates for end-users,.
*   **OLAP (Online Analytical Processing):** Analytical systems (Data Warehouses) used by analysts to run complex, long-running aggregations over massive datasets for decision support,.
*   **System of Record:** The authoritative **source of truth** for a piece of data; it holds the canonical version that is written first.
*   **Derived Data:** Data that is transformed or processed from another system (e.g., caches, search indexes, materialized views). If lost, it can be re-created from the system of record,.
*   **Cloud Native:** An architecture specifically designed to leverage cloud features like **disaggregation** (separating storage from compute).

# Architecture & Design Trade-offs

*   **Single-Node vs. Distributed Systems:**
    *   *Single-node:* Simpler to develop and reason about; avoids network unreliability,.
    *   *Distributed:* Provides **scalability** and **fault tolerance** but introduces "partial failure" where the system is half-broken and indeterminate,.
*   **Disaggregated vs. Coupled Storage:**
    *   *Traditional:* Compute and storage are on the same machine (low latency, limited scaling).
    *   *Disaggregated:* Storage (e.g., S3) and compute (transient instances) scale independently, but data must travel over the network,.
*   **Normalization vs. Denormalization:**
    *   *Normalized:* Faster writes and easier updates; requires expensive joins for reads,.
    *   *Denormalized:* Faster reads by duplicating data; more expensive and complex to keep in sync,.

# Real-World Examples

*   **Amazon S3 vs. EBS:** S3 provides distributed **object storage** for large files with high durability, while EBS emulates a **virtual block device** (disk) over the network for traditional software that expects a standard filesystem,,.
*   **GDPR Compliance:** Modern laws like the "right to be forgotten" force a technical rethink of architecture.

# Common Misconceptions

*   **"More nodes always means more speed":** In distributed systems, more nodes increase communication overhead and the probability of a "straggler" slowing down the entire request.

---

# Discussion Points

*   **Why split Operational (OLTP) and Analytical (OLAP) systems?**
*   To prevent expensive analytical queries from impacting user-facing performance and because the storage layouts (row-oriented vs. column-oriented) are fundamentally different,.
*   **What is the "Glue Code" problem?**
*   The challenge of modern systems is no longer just writing code, but the **integration** of multiple specialized tools (like a DB, a cache, and a search index) into a single reliable dataflow,.
*   **Data Outlives Code:** While application logic might be updated daily, the data in your database can persist for years, meaning schema evolution and compatibility are more critical than code versioning.

# To Remember

*   **Integration is the primary challenge:** Focus on how data flows between different specialized systems,.
*   **Abstractions hide complexity but have limits:** Tools like SQL or Cloud APIs simplify your life, but you must understand their underlying trade-offs to debug them effectively,.

---

### Personal Observation

* Before reading this chapter, I assumed that "Distributed System" was almost the same thing as "Microservices".
* What I realized is that distribution is a broader concept.

Even a traditional monolithic application can be part of a distributed system if it communicates with external components running on different machines, such as a database server, Redis, Kafka, or S3. The key idea is not the number of services, but the fact that the system as a whole spans multiple nodes that communicate over a network.

A good sanity check: if my application depends on another machine to function, I am already dealing with distributed systems concerns (network failures, latency, consistency, etc.).

If both the application and the database run on the same machine, then the system is typically not considered distributed because everything resides on a single node.

