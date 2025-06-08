# Concurrency Control Protocols in DBMS

## Definition

Concurrency Control Protocols (CCP) in a Database Management System (DBMS) are mechanisms designed to manage the simultaneous execution of multiple transactions to ensure the ACID properties (Atomicity, Consistency, Isolation, Durability), particularly isolation and consistency, as referenced in prior discussions. These protocols prevent conflicts between transactions (e.g., read-write or write-write conflicts, as discussed in conflict serializability) to maintain a consistent database state while maximizing concurrency.

## Characteristics

- **Purpose**: Ensure that concurrent transactions produce results equivalent to a serial execution (conflict serializability), preventing data inconsistencies such as dirty reads, lost updates, or non-repeatable reads.
- **Key Objectives**:
    - Maintain isolation, ensuring transactions appear to execute independently.
    - Preserve consistency by enforcing functional dependencies and constraints (referenced in prior discussions on functional dependency and normal forms).
    - Optimize performance by allowing concurrent execution without excessive blocking.
- **Relation to Prior Discussions**:
    - **Conflict Serializability**: CCPs enforce schedules that are conflict serializable, ensuring equivalence to a serial order.
    - **ACID Properties**: Focus on isolation and consistency, critical for transaction integrity.
    - **DML**: Manage conflicts arising from DML operations (`SELECT`, `UPDATE`, `INSERT`, `DELETE`).
    - **ER Model and Normalization**: Ensure that relationships and normalized schemas remain consistent under concurrent access.

## Types of Concurrency Control Protocols

Concurrency control protocols can be categorized based on their approach to managing transaction conflicts:

1. **Lock-Based Protocols**:
    
    - **Mechanism**: Use locks (e.g., shared for reads, exclusive for writes) to control access to data items.
    - **Two-Phase Locking (2PL)**:
        - **Growing Phase**: A transaction acquires all necessary locks.
        - **Shrinking Phase**: Releases all locks, ensuring no new locks are acquired after releasing any.
        - **Variants**: Strict 2PL (releases locks only at commit/rollback), Rigorous 2PL (releases all locks at commit).
        - **Advantage**: Guarantees conflict serializability.
        - **Drawback**: Can lead to deadlocks (referenced in prior discussions).
    - **Example**: Transaction ( T_1 ) locks a row in `Accounts` for an update, preventing ( T_2 ) from accessing it until ( T_1 ) commits.
2. **Timestamp-Based Protocols**:
    
    - **Mechanism**: Assign a unique timestamp to each transaction (e.g., based on start time). Access to data items is ordered by timestamps, ensuring conflict serializability.
    - **Rules**:
        - Read: Allowed if the transaction’s timestamp is greater than the write-timestamp of the data item.
        - Write: Allowed if the transaction’s timestamp is greater than both read- and write-timestamps of the data item.
        - Otherwise, the transaction is rolled back and restarted with a new timestamp.
    - **Advantage**: Deadlock-free.
    - **Drawback**: Frequent rollbacks can reduce performance.
    - **Example**: Transaction ( T_1 ) (timestamp 100) reads a data item; ( T_2 ) (timestamp 90) attempting to write is rolled back.
3. **Multiversion Concurrency Control (MVCC)**:
    
    - **Mechanism**: Maintains multiple versions of a data item, each tagged with a timestamp or transaction ID. Transactions read the version consistent with their start time.
    - **Advantage**: Enhances concurrency by allowing reads without blocking writes.
    - **Drawback**: Increased storage for maintaining versions.
    - **Example**: PostgreSQL uses MVCC to allow a transaction to read an older version of a row while another transaction updates it.
4. **Optimistic Concurrency Control**:
    
    - **Mechanism**: Assumes conflicts are rare. Transactions execute without locking, but a validation phase checks for conflicts before committing.
    - **Phases**:
        - **Read Phase**: Read and store data locally.
        - **Validation Phase**: Check if concurrent transactions conflict (e.g., using timestamps or read/write sets).
        - **Write Phase**: Apply changes if validation succeeds; otherwise, rollback.
    - **Advantage**: High concurrency in low-conflict environments.
    - **Drawback**: Rollbacks in high-conflict scenarios.
    - **Example**: Suitable for read-heavy applications like reporting systems.

## Technical Notes

- **Relation to Prior Discussions**:
    - **Conflict Serializability**: CCPs ensure schedules are conflict serializable by controlling operation order (e.g., via locks or timestamps).
    - **Functional Dependency and Normalization**: CCPs protect normalized schemas (e.g., 3NF, BCNF) from inconsistencies during concurrent updates.
    - **ER Model**: Relationships (e.g., foreign keys) require careful concurrency control to maintain referential integrity.
    - **DML**: CCPs manage conflicts in DML operations, ensuring safe execution of `UPDATE` or `DELETE`.
    - **ACID Properties**: Isolation is enforced by CCPs, while consistency aligns with functional dependencies and constraints.
- **Deadlock Handling**: Lock-based protocols may require deadlock detection and resolution (e.g., timeout or deadlock victim selection), as referenced in prior discussions on deadlocks.
- **Performance Considerations**:
    - Lock-based protocols may reduce concurrency due to blocking.
    - MVCC and optimistic protocols improve concurrency but increase storage or validation overhead.
    - Indexing (previously discussed) can complement CCPs by optimizing data access.
- **Implementation in RDBMS**:
    - MySQL (InnoDB) uses 2PL and MVCC for concurrency control.
    - PostgreSQL relies heavily on MVCC for high concurrency.
    - Refer to _MySQL Reference Manual_ ([https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)) or _PostgreSQL Documentation_ ([https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)) for details.

## Practical Project-Related Examples

### Example 1: Online Banking System

- **Project Context**: A banking system manages accounts and fund transfers, with multiple users performing concurrent transactions (e.g., transfers, balance checks).
- **ER Model Context** (from prior discussions):
    - **Entities**: `Account(AccountID, Balance)`, `Transaction(TransactionID, FromAccountID, ToAccountID, Amount)`.
    - **Relationships**: `Transfers` (links `Account` to `Transaction`).
    - **Functional Dependencies**: `AccountID → Balance`.
- **Concurrency Scenario**:
    - Transaction ( T_1 ): Transfer $100 from Account A to Account B.
        
        ```sql
        BEGIN TRANSACTION;
        UPDATE Account SET Balance = Balance - 100 WHERE AccountID = 'A';
        UPDATE Account SET Balance = Balance + 100 WHERE AccountID = 'B';
        COMMIT;
        ```
        
    - Transaction ( T_2 ): Check balance of Account A.
        
        ```sql
        BEGIN TRANSACTION;
        SELECT Balance FROM Account WHERE AccountID = 'A';
        COMMIT;
        ```
        
- **Concurrency Control**:
    - **Lock-Based (2PL)**: ( T_1 ) acquires exclusive locks on Accounts A and B, blocking ( T_2 )’s read until ( T_1 ) commits. Prevents dirty reads but may cause delays.
    - **MVCC (PostgreSQL)**: ( T_2 ) reads a version of Account A’s balance before ( T_1 )’s update, ensuring isolation without blocking.
- **Outcome**: Ensures consistent balances (e.g., no lost updates) and aligns with conflict serializability by enforcing a serial order (e.g., ( T_1 \rightarrow T_2 )).

### Example 2: E-Commerce Inventory System

- **Project Context**: An e-commerce platform manages product inventory and customer orders, with concurrent updates to stock levels during purchases.
- **ER Model Context**:
    - **Entities**: `Product(ProductID, Name, Stock)`, `Order(OrderID, CustomerID)`, `OrderItem(OrderID, ProductID, Quantity)`.
    - **Relationships**: `Contains` (many-to-many between `Order` and `Product`).
    - **Functional Dependencies**: `ProductID → Stock`.
- **Concurrency Scenario**:
    - Transaction ( T_1 ): Customer 1 purchases 5 units of Product P1.
        
        ```sql
        BEGIN TRANSACTION;
        UPDATE Product SET Stock = Stock - 5 WHERE ProductID = 'P1' AND Stock >= 5;
        INSERT INTO Order VALUES (101, 1);
        INSERT INTO OrderItem VALUES (101, 'P1', 5);
        COMMIT;
        ```
        
    - Transaction ( T_2 ): Customer 2 purchases 3 units of Product P1.
        
        ```sql
        BEGIN TRANSACTION;
        UPDATE Product SET Stock = Stock - 3 WHERE ProductID = 'P1' AND Stock >= 3;
        INSERT INTO Order VALUES (102, 2);
        INSERT INTO OrderItem VALUES (102, 'P1', 3);
        COMMIT;
        ```
        
- **Concurrency Control**:
    - **Optimistic Concurrency Control**: Both transactions read `Stock`, update locally, and validate before committing. If ( T_1 ) commits first, ( T_2 ) checks if `Stock` is still sufficient; otherwise, it rolls back.
    - **Lock-Based (Strict 2PL)**: ( T_1 ) locks `Product.P1`, blocking ( T_2 ) until ( T_1 ) commits, preventing overselling but reducing concurrency.
- **Outcome**: Prevents overselling (e.g., negative stock) and ensures consistency, leveraging normalized schema and functional dependencies.

## Revision Summary

- **Core Concept**: Concurrency Control Protocols (CCP) manage concurrent transactions to ensure conflict serializability and ACID compliance.
- **Types**: Lock-based (e.g., 2PL), timestamp-based, MVCC, and optimistic concurrency control, each balancing isolation and performance.
- **Mechanisms**: Use locks, timestamps, or versioning to prevent conflicts (e.g., dirty reads, lost updates).
- **Relevance to Prior Discussions**: Builds on conflict serializability, ACID properties, DML, normalization, and ER Model by ensuring consistent concurrent access to normalized schemas.
- **Practical Use**: Applied in banking (consistent transfers) and e-commerce (inventory management) to maintain data integrity.
- **Further Study**: Consult _Database System Concepts_ by Silberschatz, Korth, and Sudarshan for theoretical foundations, or _MySQL Reference Manual_ ([https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)) and _PostgreSQL Documentation_ ([https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)) for implementation details.