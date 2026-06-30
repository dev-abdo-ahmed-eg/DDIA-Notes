# Chapter 4 - Storage and Retrieval
---

# Chapter Mental Model
The story of Chapter 4 is about how the physical limitations of hardware (Disk and RAM) force us to design specific data structures to find our data.


# Key Ideas

*   **Evolution is Inevitable:** Applications and their data requirements change constantly due to new features, better understanding of users, or business shifts. 
*   **Compatibility is the Core Problem:** In large systems, code changes cannot happen instantaneously. We must support **backward compatibility** (new code reading old data) and **forward compatibility** (old code reading new data) to allow for rolling upgrades without downtime.
*   **Data Outlives Code:** While application code might be replaced every few minutes during a deployment, the data in the database can persist for years, often in various historical encoding versions.

# Core Concepts

*   **Encoding (Serialization/Marshaling):** The translation of in-memory data structures (optimized for CPU/pointers) into a self-contained sequence of bytes (optimized for storage or network transmission).
*   **Schema-Driven Encoding:** Formats like **Protocol Buffers** and **Avro** that require a schema to encode/decode data. They are more compact than textual formats because they omit field names from the byte stream.
*   **Field Tags:** In Protocol Buffers and Thrift, these are unique numbers assigned to fields in a schema. They are critical for compatibility: names can change, but tags must remain constant as they identify fields in the encoded data.
*   **The Writer’s and Reader’s Schema:** In Avro, the encoding process uses the version of the schema known to the writer, while the decoding process uses the reader's own version. The library resolves differences by mapping field names.
*   **Durable Execution:** A pattern provided by frameworks like **Temporal** where entire service-based workflows are executed reliably, ensuring each task runs even if machines fail.

# Design Trade-offs

*   **Textual (JSON/XML) vs. Binary Encodings:**
    *   **Textual:** Human-readable and widely supported but ambiguous with numbers (e.g., precision of large integers) and bulky because they include field names in every record.
    *   **Binary:** Much more compact and provides strict type checking and documentation via schemas, but requires a decoding step to be human-readable.
*   **Language-Specific Encodings:** Using built-in features like Java’s `java.io.Serializable` or Python’s `pickle` is generally a **bad idea**. They are often insecure, tie you to a single language, and rarely support versioning/compatibility well.
*   **Synchronous (RPC) vs. Asynchronous (Message Broker):**
    *   **RPC (REST/gRPC):** Intuitive request/response pattern but couples the availability of the client and server.
    *   **Message Brokers:** Provide better decoupling, act as a buffer for slow consumers, and can deliver a message to multiple recipients, but are more complex to set up.

# Real-World Examples

*   **Rolling Upgrades:** A system where new versions of a service are deployed to nodes one by one. This necessitates that old and new code versions coexist and communicate, requiring forward and backward compatibility.
*   **The "Unknown Field" Data Loss:** If an old version of code reads a record written by new code, it might not recognize a new field. If that old code then saves the record back to the database, it might accidentally delete the new field if the encoding library isn't designed to preserve unknown attributes.
*   **Avro for Archival Storage:** Snapshots of databases are often stored in Avro object container files or columnar formats like Parquet, providing a consistent, schema-documented version of the data for long-term analytics.

# Common Misconceptions

*   **"Binary JSON is always better":** While formats like BSON or MessagePack are binary, they still include all field names in the encoded data. They are only slightly more compact than JSON; true efficiency gains come from **schema-driven** binary formats like Protobuf or Avro.
*   **Relying on the Database to Handle Migrations:** While relational databases can perform schema changes (e.g., `ALTER TABLE`), they often do so asynchronously or through expensive rewrites. The application code must still be able to handle "dirty" data that doesn't yet conform to the new schema.
*   **Assuming gRPC is "just faster REST":** While gRPC uses efficient Protocol Buffers, it introduces a different mental model regarding service discovery, load balancing, and network failure modes compared to REST over HTTP.

# Discussion Points

*   **How does Avro handle schema evolution without field tags?** It relies on the reader having both the writer's schema (often found in a file header or registry) and its own schema, mapping fields by name and ignoring or filling in defaults for missing items.
*   **What does "Data outlives code" imply for your deployment strategy?** It means you cannot rely on code-level migrations to "fix" all your data; your backend must be designed to gracefully handle records written by code versions from years ago.
*   **Why is the "Outbox Pattern" used in microservices?** It allows a service to update its database and send a message (e.g., to Kafka) within a single local transaction, avoiding the need for expensive and unreliable distributed transactions.

# What I Must Remember 

*   **Design for Rolling Upgrades:** Never assume all nodes are running the same version of code. Always verify that your encoding format and application logic support **forward and backward compatibility**.
*   **Schemas are Documentation:** A machine-readable schema is a "source of truth" that is guaranteed to be up-to-date, unlike static text documentation.
*   **Preserve Unknown Fields:** When data flows through a process (like a database update or a message republisher), ensure it doesn't "clobber" fields it doesn't recognize.