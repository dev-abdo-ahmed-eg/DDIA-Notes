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
