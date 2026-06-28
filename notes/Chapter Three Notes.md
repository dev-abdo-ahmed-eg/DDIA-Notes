# CHAPTER 3 - Data Models and Query Languages

---

# Key Ideas

* **Data Models Shape Thought:** Data models are one of the most important parts of software design because they influence how we model problems and structure applications.
* **Layered Abstractions:** Applications are built on layers of abstraction (Application Objects → JSON/Tables → Storage Engines → Hardware). Each layer hides implementation details and enables independent innovation.
* **No One-Size-Fits-All:** Every data model is optimized for different access patterns. Choosing the right model depends on how the application reads and writes data.
* **Convergence of Database Models:** Modern databases are becoming multi-model. Relational databases now support JSON and graph features, while document databases support joins, indexes, and declarative query languages.

---

# Core Concepts

## Relational Model

* Organizes data into relations (tables) and tuples (rows).
* Based on Edgar Codd's relational model (1970).
* Best suited for structured data and complex relationships.

## Document Model

* Stores self-contained documents (typically JSON).
* Maps naturally to application objects.
* Excels at hierarchical one-to-many relationships.

## Graph Model

* Represents data as **vertices (nodes)** and **edges (relationships)**.
* Ideal for highly connected domains where relationships are first-class citizens.

## Normalization

* Store each fact only once.
* Reduces redundancy.
* Improves consistency.
* Requires joins when reading.

## Denormalization

* Duplicate data to optimize reads.
* Reduces joins.
* Increases storage usage.
* Makes updates more expensive because multiple copies must remain consistent.

## Schema-on-Write vs. Schema-on-Read

**Schema-on-Write (Relational Databases)**

* Schema enforced before writing.
* Strong consistency.
* Easier collaboration across teams.

**Schema-on-Read (Document Databases)**

* Schema interpreted by the application.
* More flexibility.
* Better for heterogeneous or evolving data.

## Declarative Query Languages

Declarative languages describe **what** data is required instead of **how** to retrieve it.

Examples:

* SQL
* Cypher
* SPARQL

This allows the database optimizer to choose the most efficient execution plan.

---

# Architecture & Design Trade-offs

## Document vs. Relational

### Document Model Advantages

* Better data locality.
* Fewer joins.
* Natural representation of hierarchical data.

### Relational Model Advantages

* Better support for many-to-many relationships.
* Mature query optimization.
* Strong consistency guarantees.

---

## Embedding vs. References

Embed related data when:

* It naturally belongs together.
* It is usually retrieved together.
* The embedded collection remains reasonably small.

Use references when:

* Relationships are many-to-many.
* Collections may grow without bound (for example, thousands of comments).
* Related entities are shared across multiple documents.

---

## Normalization vs. Denormalization

| Normalized         | Denormalized       |
| ------------------ | ------------------ |
| Faster writes      | Faster reads       |
| Less duplication   | Duplicate data     |
| Requires joins     | Fewer joins        |
| Easier consistency | Higher update cost |

There is no universally correct choice. Optimize according to your application's workload.

---

## Schema Evolution

**Schema-on-Read**

* Easier evolution
* Better for dynamic data

**Schema-on-Write**

* Better consistency
* More difficult migrations

---

## Event Sourcing

Instead of storing only the latest state:

* Every state change is stored as an immutable event.
* Current state is derived from the event history.
* Naturally provides an audit log.
* Typically offers high write throughput because events are appended sequentially.

---

# Real-World Examples

## LinkedIn Profile

A profile is naturally represented as a document:

* Personal information
* Education
* Work experience

However, companies and universities introduce many-to-many relationships that may require references.

---

## Facebook Social Graph

Graph databases naturally model:

* Users
* Friendships
* Comments
* Likes
* Events

where relationships are the primary concern.

---

## Conference Booking System

A booking system benefits from Event Sourcing because every reservation, cancellation, and payment becomes a permanent event.

Current availability can always be recomputed.

---

## Movie Recommendation Systems

Relational data is often transformed into sparse matrices before applying machine learning algorithms for recommendations.

---

# Common Mistakes & Misconceptions

## "NoSQL Means Schemaless"

False.

Every application has a schema.

The difference is whether the schema is enforced by:

* the database (Schema-on-Write), or
* the application (Schema-on-Read).

---

## "Joins Don't Scale"

Not necessarily.

Many large-scale systems successfully perform joins (or application-level joins) because the workload can be parallelized efficiently.

---

## "ORMs Eliminate the Object-Relational Mismatch"

ORMs reduce boilerplate but cannot completely hide the differences between objects and relational tables.

Poor ORM usage often introduces the N+1 Query Problem.

---

# Interview / Discussion Points

* What is the Object-Relational Impedance Mismatch?
* Explain the N+1 Query Problem and how to avoid it.
* When should you choose a document database instead of a relational database?
* When is a graph database the better choice?
* What are the trade-offs between normalization and denormalization?
* Explain Schema-on-Write vs. Schema-on-Read.
* Why are declarative query languages preferred?
* What problems do Event Sourcing and CQRS solve?

---

# Design Principles

* Data usually outlives application code.
* Design your data model around access patterns, not trends.
* Abstractions reduce complexity by hiding lower-level implementation details.
* There is no perfect data model—every choice involves trade-offs.
* Optimize for the operations your application performs most frequently.

---

# Key Insights

These are practical engineering lessons worth remembering beyond the chapter summary.

## SQL is Declarative

SQL specifies **what** data should be returned rather than **how** to retrieve it.

The database optimizer decides:

* execution plan
* indexes
* join algorithms

This separation allows the same SQL query to become faster as the optimizer improves.

---

## The N+1 Query Problem

A common ORM performance issue:

1. Query N records.
2. Execute one additional query for each record to load related data.

Result:

* N + 1 database queries
* Poor performance

Solutions include:

* joins
* eager loading
* batch fetching

---

## Documents Can Simplify Reads

Embedding related data inside a document often allows retrieving everything with a single query.

This reduces:

* joins
* query complexity
* application logic

Use this approach only when the embedded data naturally belongs together.

---

## Modern Document Databases Support More Than Key-Value Access

Document databases such as MongoDB now support:

* declarative query languages
* aggregation pipelines
* secondary indexes
* joins (`$lookup`)

The gap between relational and document databases continues to shrink.

---

## When a Schema-less Model Fits Best

A document database is often appropriate when:

* records have different structures,
* many object types exist,
* or the schema is determined by external systems.

---

# Further Exploration

This chapter introduced several concepts that extend beyond database design and deserve deeper study.

## GraphQL

* Client-driven query language.
* Clients request exactly the fields they need.
* Reduces over-fetching and under-fetching.
* Not intended for arbitrary search conditions like SQL.

## Event Sourcing

* Every state change is stored as an immutable event.
* Events become the source of truth.

## CQRS (Command Query Responsibility Segregation)

* Separate write models from read models.
* Maintain read-optimized views derived from write operations.
* Frequently combined with Event Sourcing but also valuable independently.

These topics appear frequently in modern distributed systems and backend architectures and are worth revisiting after completing the book.
