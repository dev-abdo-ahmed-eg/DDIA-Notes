# CHAPTER 3 - Data Models and Query Languages
---

# Key Ideas

*   **Data Models Shape Thought:** Data models are the most critical part of software development because they dictate not only how code is written but also how we perceive and solve problems.
*   **Layered Abstractions:** Applications are built by layering models (e.g., Application Objects $\rightarrow$ General-purpose models like JSON/Tables $\rightarrow$ Storage engines $\rightarrow$ Hardware). Each layer hides complexity to allow different engineering groups to work together.
*   **The "No One-Size-Fits-All" Rule:** Different models excel at different access patterns. The relational model remains a standard for many, but document and graph models are better for specific types of data relationships.
*   **Convergence:** Databases are becoming more multi-model; relational databases are adding JSON/Graph support, and document databases are adding joins and declarative queries.

# Core Concepts

*   **Relational Model:** Organizes data into relations (tables) and tuples (rows), based on a model first proposed by Edgar Codd in 1970.
*   **Document Model:** Stores data in self-contained documents (often JSON), which maps closely to application-level objects and excels at one-to-many "tree" structures.
*   **Graph Model:** Consists of **vertices** (nodes/entities) and **edges** (relationships). It is designed for highly interconnected data where "anything can be related to everything".
*   **Normalization:** Storing a fact in exactly one place and referencing it by ID. This reduces redundancy and makes updates easier but requires **joins** for reading.
*   **Denormalization:** Duplicating information (like storing a string instead of an ID) to speed up reads by avoiding joins, at the cost of more expensive writes and potential inconsistency.
*   **Schema-on-Write vs. Schema-on-Read:** 
    *   *Schema-on-write:* The database enforces structure when data is inserted (relational).
    *   *Schema-on-read:* The structure is interpreted by the application when data is accessed (document/NoSQL).
*   **Declarative Query Language:** Specifies the *pattern* of data wanted, not *how* to find it. This allows the database optimizer to decide the most efficient execution path (e.g., SQL, Cypher, SPARQL).
*   **Event Sourcing:** Treats a log of immutable events as the system of record, deriving the current state (materialized views) through a process like **CQRS**.

# Architecture & Design Trade-offs

*   **Locality vs. Flexibility:** Document models offer **data locality** (fetching everything in one place) for one-to-many structures, but they struggle with many-to-many relationships where normalized relational models or graphs perform better.
*   **Join Performance:** While relational models excel at joins, performing "joins" in application code (hydrating IDs) for document models can be easier to scale and parallelize for certain workloads.
*   **Schema Evolution:** Schema-on-read is better for **heterogeneous data** (where records have different structures), whereas schema-on-write ensures consistency across teams but makes large-scale migrations operationally challenging.
*   **Write Throughput:** Append-only event logs (Event Sourcing) typically provide higher write throughput due to sequential access patterns compared to databases that update data in place.

# Real-World Examples

*   **LinkedIn Profiles:** A profile is a natural tree structure (one user, multiple jobs/education), but references to companies and schools create many-to-many relationships that challenge a pure document model.
*   **Facebook Social Graph:** Uses a graph model where vertices are people, events, and comments, and edges represent friendships, check-ins, or replies.
*   **Conference Bookings:** A complex domain where calculating available seats is difficult; using an immutable event log (Event Sourcing) allows you to recompute state and maintain an audit trail.
*   **Movie Recommendations:** Relational rating data is often transformed into a **sparse matrix** (DataFrames) for machine learning algorithms to compute recommendations via linear algebra.

# Common Mistakes & Misconceptions

*   **"NoSQL is schemaless":** This is a myth. There is always an **implicit schema** that the application code assumes when reading; the difference is whether that schema is enforced by the database (write) or the app (read).
*   **"Joins are bad for scale":** High-performance systems like X (Twitter) still use joins (hydrating IDs in application code) effectively because the cost is parallelizable and predictable.
*   **"ORMs solve the model mismatch":** Object-Relational Mappers reduce boilerplate but cannot fully hide the differences between relational and object models, often leading to the **N+1 query problem**.

# Interview / Discussion Points

*   **What is Impedance Mismatch?** The disconnect between the rich object-oriented structures in code and the rigid tables/rows of a relational database.
*   **Explain the N+1 Query Problem:** A common ORM inefficiency where one query for a list of items leads to $N$ additional queries to fetch related data for each item.
*   **When would you choose a Graph Model over Relational?** When relationships are many-to-many and queries are **recursive** (e.g., "finding friends-of-friends"), which is awkward and slow in standard SQL.

# What To Remember

*   **Data outlives code:** Application features and code change weekly, but data in your database can persist for years, making schema evolution more critical than code versioning.
*   **Model for the query pattern:** The best data model is the one that makes your most frequent and critical queries simple and fast.
*   **Abstractions save us from hardware:** Each layer of the data model is an abstraction that allows us to focus on higher-level logic without worrying about bytes on a disk or electrical currents.