# Database - Things found on the Internet

## Database essentials
👉 ACID properties
- Atomicity: Transactions are binary; they either complete fully or don’t execute at all.
- Consistency: Post any transaction, the database remains consistent.
- Isolation: Each transaction operates independently.
- Durability: Once data is committed, it’s there for the long haul.

👉 Vertical Scaling (“Scale Up”) vs Horizontal Scaling (“Scale Out”)

👉 Database Sharding: This involves distributing different data segments (or shards) across multiple servers, improving performance, scalability, and ease of management.
- Range-based Sharding: Based on a specific key’s range.
- Directory-based Sharding: A lookup service directs traffic.
- Geographical Sharding: Data distribution based on location.

👉 Replication: It’s all about duplicating data across servers for high availability.
- Master-Slave Replication: One primary database with numerous read-only replicas.
- Master-Master Replication: Multiple read-write databases.

👉 Boosting Database Performance
- Caching: Leverage caching to store frequent queries, using in-memory databases like Redis, enhancing performance manifold.
- Indexing: Much like a book’s index, database indexes on **often-accessed columns** can drastically improve retrieval times.
- Query Optimization: Streamline your queries, minimize joins, and always, always steer clear of the generic SELECT *.

👉 The CAP (Consistency, Availability, and Partition tolerance) Theorem: A Pillar in Database Decision Making

## Wide-column DB vs Column family DB vs Columnar DB vs Column-oriented DB 🫠
First, **Columnar DB** and **Column-oriented DB** essentially refer to the same concept in the context of database design. In terms of storage, they will store each column separately on disk. These databases are best suited for **OLAP (Online analytical processing)** use cases.

Examples:
- Google BigQuery
- AWS Redshift

Next, we have **Column family DB** that store data in Column families. In fact, **a Column family** is like **a Table** in a relational database. **A Column family** can have multiple rows - each row has a **unique row key identifier** and can have different number of columns than other rows. It is important to know that Column-family databases **don't support joins**.

Furthermore, when a column consists of a map of columns, then we have a **Super Column**. Think of a **Super Column** as a container of columns. And when we use **Super Columns** to create a Column family, we get a **Super Column family**.

Examples:
- Google BigTable
- Cassandra
- Druid
- etc

Lastly, a **Wide-column DB** is **a type of** Columnar DB that uses Column families.

## Different file formats (in BigData 👀)
Without being exhaustive, here are some popular File formats
- CSV
- JSON
- XML
- Protocol Buffers
    - Schema is required to interpret the data
    - Not compressible
- Avro
    - Row-based data storage format
    - Store schema in JSON and data in binary format
    - Easily handle schema evolution
    - Splittable and compressible
- Parquet
    - Column-oriented data storage format
    - Metadata includes file schema and structure
    - Supports complex data types
- ORC (Optimized Row Columnar)
    - Column-oriented data storage format
    - Does not support schema evolution (e.g recreate the file to add new data)
    - Highly-efficient way to store Apache Hive data

## Data Serialization 🔗
**Serialisation** is the process of taking an in-memory data structure (like a data frame), and converting it into a sequence of bytes. Those bytes can either be written to disk (when you’re saving a file) or they can be transmitted over some other channel.

Regardless of what you want to do with the serialised data, this conversion takes time and resources, and at some point the data will need to be **unserialised** later. The resources expended in doing so are referred to as the **Serialisation Overhead**.

## Databases: [Workload & Deployment](https://betterprogramming.pub/duckdb-whats-the-hype-about-5d46aaa73196) 🗃️
Current databases are optimized for analytical (OLAP) or transactional workloads (OLTP). Besides, current database technologies are deployed as **Stand-alone** or **Embedded** solutions. **Stand-alone** databases are typically deployed in a client-server paradigm. The database sits on a centralized server and is queried by a client application. [**Embedded** databases](https://en.wikipedia.org/wiki/Embedded_database) run within the host process of whatever application is accessing the database.

| Workload  | Transactional (OLTP) | Analytical (OLAP) |
| ------------- | ------------- | ------------- |
| Deployment  |   |   |
| Stand-alone  | PostgreSQL, MySQL  | Snowflake, ClickHouse, Redshift, BigQuery  |
| Embedded  | SQLite, SolidDB  | DuckDB  |

## Schema used in Data Warehouse
- Star ⭐ Schema  
    - simple, designed to be easy to understand
    - create a denormalized tables - introduce redundancy for improved query performance
    - higher disk space due to data redundancy
- Snowflake ❄️ Schema 
    - more complicated compared to Star Schema
    - create normalized tables - eliminate any redundant data to reduce overhead
    - lower disk space due to limited data redundancy
- Galaxy 🌌 Schema (aka Fact Constellation Schema)
    - create normalized tables
    - reduces redundancy to near zero redundancy as a result of normalization


