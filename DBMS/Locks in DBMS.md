## Definition

Locks in a Database Management System (DBMS) are mechanisms used to control access to database resources (e.g., tables, rows, or data pages) during concurrent transaction execution. They ensure that multiple transactions can operate safely on shared data without causing conflicts, thereby maintaining data consistency and supporting the ACID properties (Atomicity, Consistency, Isolation, Durability), as referenced in prior discussions on concurrency control protocols and conflict serializability.

## Why Locks Are Needed

Locks are essential in a DBMS to manage concurrency in multi-user environments, where multiple transactions access and modify entity sets (as discussed in entity set notes) simultaneously. Without locks, concurrent transactions could lead to:

- **Data Inconsistencies**: Issues like dirty reads (reading uncommitted data), non-repeatable reads, phantom reads, or lost updates, as referenced in conflict serializability discussions.
- **Violation of ACID Properties**: Particularly isolation (transactions appearing to execute independently) and consistency (maintaining functional dependencies and constraints).
- **Conflicts**: Read-write or write-write conflicts between transactions, disrupting conflict serializability.

Locks prevent these issues by restricting access to data resources, ensuring that transactions execute in a controlled manner, aligning with the ER Model’s relationships and normalized schemas (e.g., 3NF, as discussed in normal forms).

## Practical Examples

### Example 1: Banking System

- **Context**: A banking system (referenced in prior concurrency control notes) manages accounts with transactions like fund transfers, requiring locks to prevent lost updates.
- **Schema**:
    
    ```sql
    CREATE TABLE Accounts (
        AccountID VARCHAR(10) PRIMARY KEY,
        Balance DECIMAL(10,2)
    );
    INSERT INTO Accounts VALUES ('A1', 1000.00), ('A2', 500.00);
    ```
    
- **Scenario**:
    - Transaction ( T_1 ): Transfer $100 from Account A1 to A2.
        
        ```sql
        BEGIN TRANSACTION;
        UPDATE Accounts SET Balance = Balance - 100 WHERE AccountID = 'A1';
        UPDATE Accounts SET Balance = Balance + 100 WHERE AccountID = 'A2';
        COMMIT;
        ```
        
    - Transaction ( T_2 ): Check balance of Account A1.
        
        ```sql
        SELECT Balance FROM Accounts WHERE AccountID = 'A1';
        ```
        
- **Lock Usage**:
    - ( T_1 ) acquires an _exclusive lock_ on rows for A1 and A2 to prevent concurrent modifications.
    - ( T_2 ) requests a _shared lock_ for reading A1 but is blocked until ( T_1 ) releases its exclusive lock after committing.
- **Outcome**: Prevents ( T_2 ) from reading an inconsistent balance (e.g., after A1’s debit but before A2’s credit), ensuring isolation and consistency.

### Example 2: E-Commerce Inventory System

- **Context**: An e-commerce system (referenced in prior concurrency control notes) manages product stock with concurrent purchase transactions.
- **Schema**:
    
    ```sql
    CREATE TABLE Products (
        ProductID VARCHAR(10) PRIMARY KEY,
        Name VARCHAR(50),
        Stock INT
    );
    INSERT INTO Products VALUES ('P1', 'Laptop', 10);
    ```
    
- **Scenario**:
    - Transaction ( T_1 ): Customer 1 buys 3 units of Product P1.
        
        ```sql
        BEGIN TRANSACTION;
        UPDATE Products SET Stock = Stock - 3 WHERE ProductID = 'P1' AND Stock >= 3;
        COMMIT;
        ```
        
    - Transaction ( T_2 ): Customer 2 buys 5 units of Product P1.
        
        ```sql
        BEGIN TRANSACTION;
        UPDATE Products SET Stock = Stock - 5 WHERE ProductID = 'P1' AND Stock >= 5;
        COMMIT;
        ```
        
- **Lock Usage**:
    - ( T_1 ) acquires an _exclusive lock_ on the P1 row to update stock.
    - ( T_2 ) waits for the lock to be released, preventing overselling (e.g., stock becoming negative).
- **Outcome**: Ensures accurate stock levels, maintaining consistency and preventing conflicts.

## Role of Locks in Implementing ACID Properties

Locks are critical for enforcing ACID properties, particularly:

- **Atomicity**: Locks ensure that a transaction’s operations (e.g., multiple `UPDATE` statements) are completed as a single unit. For example, in the banking system, locks on both accounts prevent partial updates if a transaction fails.
- **Consistency**: Locks enforce constraints (e.g., foreign keys, functional dependencies) by preventing concurrent modifications that could violate them, as seen in the inventory system’s stock updates.
- **Isolation**: Locks provide transaction isolation by controlling access to data, ensuring that transactions appear to execute sequentially (conflict serializability). For instance, shared locks prevent dirty reads in the banking example.
- **Durability**: While locks do not directly ensure durability (persistence of committed data), they support it indirectly by ensuring consistent states before commits, which are then made durable.

Locks work with Transaction Control Language (TCL) commands (e.g., `COMMIT`, `ROLLBACK`, as discussed in SQL commands) and concurrency control protocols (e.g., two-phase locking) to enforce these properties.

## Difference Between Shared Lock and Exclusive Lock

Shared and exclusive locks are the primary types of locks used in DBMS to manage concurrent access. The differences are outlined below:

|**Aspect**|**Shared Lock (S-Lock)**|**Exclusive Lock (X-Lock)**|
|---|---|---|
|**Definition**|Allows multiple transactions to read a resource simultaneously but prevents writing.|Allows a single transaction to read and write a resource, preventing all other access.|
|**Purpose**|Ensures read consistency without blocking other reads.|Ensures write consistency by preventing any concurrent access.|
|**Compatibility**|Compatible with other shared locks; multiple transactions can hold shared locks on the same resource.|Incompatible with all locks (shared or exclusive); only one transaction can hold an exclusive lock.|
|**Use Case**|Reading data (e.g., `SELECT` queries), as in balance checks.|Modifying data (e.g., `UPDATE`, `INSERT`, `DELETE`), as in fund transfers or stock updates.|
|**Example Operation**|`SELECT Balance FROM Accounts WHERE AccountID = 'A1';`|`UPDATE Accounts SET Balance = Balance - 100 WHERE AccountID = 'A1';`|
|**Concurrency Impact**|High concurrency for reads, as multiple transactions can read simultaneously.|Low concurrency, as it blocks all other transactions (read or write).|

- **Example** (from Banking System):
    - ( T_2 ) requests a _shared lock_ to read A1’s balance, allowing other transactions to read A1 concurrently.
    - ( T_1 ) requests an _exclusive lock_ to update A1 and A2, blocking all reads and writes until it commits, ensuring no concurrent modifications interfere.

## Different Locking Algorithms

Locking algorithms define how locks are acquired and released to ensure concurrency control. The primary algorithms are:

1. **Two-Phase Locking (2PL)**:
    - **Mechanism**: Transactions follow two phases:
        - **Growing Phase**: Acquire all necessary locks (shared or exclusive) without releasing any.
        - **Shrinking Phase**: Release all locks without acquiring new ones.
    - **Variants**:
        - **Strict 2PL**: Releases locks only at commit or rollback, ensuring strict schedules.
        - **Rigorous 2PL**: Releases all locks at commit, simplifying recovery.
    - **Advantage**: Guarantees conflict serializability (as discussed in conflict serializability notes).