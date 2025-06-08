Below is a concise, clear, and technical set of notes on **Keys in DBMS**, structured for direct use as revision material. The content adheres to a formal tone, avoids colloquialisms, and is tailored for quick revision, as per the user’s requirements. References to prior discussions (e.g., DBMS, database, database system, RDBMS, ACID properties, database languages, vertical/horizontal scaling, sharding, processes, threads, virtual memory, paging, demand paging, segmentation, thrashing, fragmentation, deadlocks, starvation, spooling, semaphores, mutexes, Belady’s Anomaly, real-time operating systems, main memory, secondary memory, dynamic binding, FCFS, SJF, SRTF, LRTF, Priority, Round Robin scheduling, Producer-Consumer Problem, Banker’s Algorithm, cache, direct/associative mapping, multitasking/multiprocessing) are noted where applicable, without rewriting previously covered content. The notes are presented with up-to-date context, considering the current date and time (02:19 PM IST, Sunday, June 08, 2025).

---

# Keys in DBMS

## Definition
In a **Database Management System (DBMS)** (*DBMS*, *RDBMS*), a **key** is a field or combination of fields in a database table that uniquely identifies records, enforces data integrity, or establishes relationships between tables. Keys are essential for efficient data retrieval, maintaining **data integrity** (*ACID Properties*), and supporting relational structures in a **database** (*Database*, *Database System*).

## Types of Keys in DBMS

1. **Candidate Key**:
   - **Definition**: A minimal set of attributes that can uniquely identify each row in a table (i.e., no subset of the candidate key can uniquely identify rows).
   - **Characteristics**:
     - Multiple candidate keys may exist in a table.
     - Ensures **entity integrity** by guaranteeing uniqueness (*RDBMS*).
     - One candidate key is chosen as the **primary key**.
   - **Example**: In a `Students` table, `StudentID` and `Email` (if unique) are candidate keys.
   - **Relation to Prior Discussions**: Enforced via **Data Definition Language (DDL)** constraints (*Database Languages*), stored in secondary memory (*Main Memory and Secondary Memory*).

2. **Primary Key**:
   - **Definition**: A candidate key selected to uniquely identify each row in a table.
   - **Characteristics**:
     - Must be **unique** and **non-null** for every record.
     - Only one primary key per table.
     - Used for indexing to optimize queries (*Cache*, *Direct/Associative Mapping*).
     - Enforces **entity integrity** (*ACID Properties*).
   - **Example**: `StudentID` as the primary key in a `Students` table.
   - **Relation to Prior Discussions**: 
     - Ensures consistency in transactions (*ACID Properties*).
     - Impacts query performance via caching (*Cache*).
     - May be used as a **shard key** in distributed systems (*Sharding*).

3. **Foreign Key**:
   - **Definition**: A field (or set of fields) in one table that references the primary key (or candidate key) of another table, establishing a relationship between tables.
   - **Characteristics**:
     - Enforces **referential integrity**, ensuring valid references (*RDBMS*).
     - Can be null (optional relationship) or non-null (mandatory).
     - Supports table joins in queries (*Database Languages*).
   - **Example**: In an `Orders` table, `CustomerID` is a foreign key referencing `CustomerID` (primary key) in a `Customers` table.
   - **Relation to Prior Discussions**:
     - Maintains consistency across tables (*ACID Properties*).
     - Concurrent updates risk deadlocks (*Deadlock*, *Semaphore and Mutex*).
     - Impacts distributed systems with sharding (*Sharding*).

4. **Super Key**:
   - **Definition**: A set of one or more attributes that can uniquely identify rows in a table (not necessarily minimal).
   - **Characteristics**:
     - Superset of candidate keys; includes additional attributes.
     - Less efficient than candidate keys due to redundancy.
   - **Example**: In a `Students` table, `{StudentID, Name}` is a super key (but `StudentID` alone is sufficient).
   - **Relation to Prior Discussions**: Rarely used directly but relevant for understanding key hierarchies (*Database*).

5. **Composite Key (Compound Key)**:
   - **Definition**: A key consisting of two or more attributes that together uniquely identify each row when no single attribute is sufficient.
   - **Characteristics**:
     - Used when a single attribute cannot ensure uniqueness.
     - Acts as a primary or candidate key if minimal.
   - **Example**: In a `ClassEnrollment` table, `{StudentID, CourseID}` as a composite key identifies unique enrollments.
   - **Relation to Prior Discussions**:
     - Impacts indexing and query performance (*Cache*).
     - Used in sharding for complex shard keys (*Sharding*).

6. **Unique Key**:
   - **Definition**: A set of attributes that ensures uniqueness for each row but allows null values (unlike primary key).
   - **Characteristics**:
     - Can have multiple unique keys per table.
     - Enforces uniqueness but not necessarily used for relationships.
   - **Example**: In a `Students` table, `Email` as a unique key (can be null for some students).
   - **Relation to Prior Discussions**:
     - Enforced via DDL constraints (*Database Languages*).
     - Supports data integrity (*ACID Properties*).

7. **Alternate Key**:
   - **Definition**: A candidate key that is not selected as the primary key.
   - **Characteristics**:
     - Could serve as the primary key but is used for other purposes (e.g., unique constraints).
   - **Example**: If `StudentID` is the primary key, `Email` (a candidate key) is an alternate key.
   - **Relation to Prior Discussions**: Similar to unique key, impacts query optimization (*Cache*).

8. **Surrogate Key** (Optional):
   - **Definition**: An artificial key (e.g., auto-incremented ID) used as the primary key, unrelated to the data’s natural attributes.
   - **Characteristics**:
     - Simplifies relationships and indexing.
     - Not derived from data, reducing dependency on business logic.
   - **Example**: An auto-incremented `ID` in a `Users` table instead of using `Email`.
   - **Relation to Prior Discussions**:
     - Common in distributed databases for sharding (*Sharding*).
     - Improves query performance (*Cache*).

### Technical Notes
- **Purpose**:
  - Ensure **data integrity** (entity and referential, *ACID Properties*).
  - Enable efficient data retrieval via indexing (*Cache*, *Direct/Associative Mapping*).
  - Support relationships in relational databases (*RDBMS*, *Database*).
  - Facilitate distributed data management (*Sharding*, *Vertical and Horizontal Scaling*).
- **Implementation**:
  - Defined using **Data Definition Language (DDL)** (e.g., `PRIMARY KEY`, `FOREIGN KEY`, *Database Languages*).
  - Stored in database metadata in secondary memory (*Main Memory and Secondary Memory*).
  - Indexed for fast access, cached in main memory (*Cache*).
  - Managed by DBMS processes/threads (*Processes and Threads*, *Multitasking and Multiprocessing*).
- **Relation to Prior Discussions**:
  - **Database/DBMS/RDBMS/Database System** (*Database*, *DBMS*, *RDBMS*, *Database System*): Keys are fundamental to relational database design and integrity.
  - **ACID Properties** (*ACID Properties*): Keys enforce consistency and referential integrity.
  - **Database Languages** (*Database Languages*): Keys are defined/modified via DDL, queried via DML.
  - **Vertical and Horizontal Scaling** (*Vertical and Horizontal Scaling*): Primary/foreign keys impact sharding strategies (*Sharding*).
  - **Main Memory and Secondary Memory** (*Main Memory and Secondary Memory*): Keys and indexes are cached for performance (*Cache*).
  - **Cache** (*Cache*, *Direct/Associative Mapping*): Keys optimize query execution via indexing.
  - **Virtual Memory/Paging/Demand Paging** (*Virtual Memory*, *Paging*, *Demand Paging*): Large key indexes risk thrashing (*Thrashing*, *Fragmentation*).
  - **Processes/Threads** (*Processes and Threads*): Key constraints are enforced by DBMS threads (*Multitasking and Multiprocessing*).
  - **Semaphore and Mutex** (*Semaphore and Mutex*): Locks on keys prevent concurrent conflicts (*Producer-Consumer Problem*).
  - **Deadlock** (*Deadlock*, *Banker’s Algorithm*): Key updates in transactions risk deadlocks.
  - **Starvation** (*Starvation and Aging*): Long transactions updating keys may starve others (*Priority Scheduling*).
  - **Spooling** (*Spooling*): Query queues involving keys resemble spooling (*Producer-Consumer Problem*).
  - **Real-Time Operating System** (*Real-Time Operating System*): Keys in real-time DBMS ensure fast, deterministic access.
  - **Dynamic Binding** (*Dynamic Binding*): Dynamic queries reference keys for flexibility.
  - **Scheduling** (*FCFS Scheduling*, *SJF Scheduling*, *SRTF Scheduling*, *LRTF Scheduling*, *Priority Scheduling*, *Round Robin Scheduling*): Query execution with keys uses scheduling.
- **Performance**:
  - Keys improve query speed via indexing but increase storage and update overhead (*Cache*, *Main Memory and Secondary Memory*).
  - Foreign key checks add latency in concurrent systems (*Semaphore and Mutex*).
  - Efficient in 2025 with NVMe SSDs reducing disk latency (*Main Memory and Secondary Memory*).
- **Modern Context**:
  - In 2025, keys remain critical for RDBMS (e.g., PostgreSQL, Oracle) and distributed databases (e.g., CockroachDB).
  - Surrogate keys are common in cloud-native systems for sharding (*Sharding*).
  - NoSQL databases (*DBMS*) use analogous key structures (e.g., partition keys in DynamoDB).

### Example
- **Scenario**: A library database in MySQL (*RDBMS*).
  - **Primary Key**: `BookID` in `Books` table uniquely identifies each book.
  - **Foreign Key**: `BookID` in `Loans` table references `Books(BookID)` to track borrowed books.
  - **Candidate Key**: `ISBN` in `Books` (alternate unique identifier).
  - **Composite Key**: `{MemberID, BookID}` in `Loans` uniquely identifies each loan.
  - **Unique Key**: `MemberEmail` in `Members` ensures unique emails.
  - **Context**: Keys enforce integrity (*ACID Properties*), optimize queries (*Cache*), and support sharding by `MemberID` (*Sharding*).

---

## Revision Summary
- **Keys in DBMS**:
  - Attributes identifying records, ensuring integrity, and linking tables.
  - Types: Candidate, Primary, Foreign, Super, Composite, Unique, Alternate, Surrogate.
- **Context**:
  - Relates to **database** (*Database*), **DBMS** (*DBMS*), **RDBMS** (*RDBMS*), **database system** (*Database System*), **ACID properties** (*ACID Properties*), **database languages** (*Database Languages*), **vertical/horizontal scaling** (*Vertical and Horizontal Scaling*), **sharding** (*Sharding*), **main/secondary memory** (*Main Memory and Secondary Memory*), **cache** (*Cache*, *Direct/Associative Mapping*), **virtual memory/paging** (*Virtual Memory*, *Paging*, *Demand Paging*), **thrashing** (*Thrashing*), **fragmentation** (*Fragmentation*), **processes/threads** (*Processes and Threads*), **semaphores/mutexes** (*Semaphore and Mutex*), **deadlocks** (*Deadlock*, *Banker’s Algorithm*), **starvation** (*Starvation and Aging*), **spooling** (*Spooling*), **Producer-Consumer** (*Producer-Consumer Problem*), **RTOS** (*Real-Time Operating System*), **dynamic binding** (*Dynamic Binding*), **scheduling** (*FCFS Scheduling*, *SJF Scheduling*, *SRTF Scheduling*, *LRTF Scheduling*, *Priority Scheduling*, *Round Robin Scheduling*), **multitasking/multiprocessing** (*Multitasking and Multiprocessing*).

---

These notes are concise for quick revision, with technical details and references to prior discussions. They can be used directly for studying database or operating system concepts or preparing for exams. For further details, refer to DBMS textbooks (e.g., "Database System Concepts" by Silberschatz) or documentation (e.g., MySQL, PostgreSQL manuals).