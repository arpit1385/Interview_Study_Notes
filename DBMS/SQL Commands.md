# SQL Commands in DBMS

## Definition

Structured Query Language (SQL) commands are instructions used to interact with a Relational Database Management System (RDBMS) to define, manipulate, control, and query data. SQL is a standardized language for managing relational databases, enabling operations on entity sets, relationships, and schemas as defined in the Entity-Relationship (ER) Model (referenced in prior discussions). SQL commands are categorized based on their functionality, supporting tasks such as schema creation, data manipulation, access control, and transaction management.

## Characteristics

- **Purpose**: Facilitates interaction with the database to perform operations like creating tables, inserting data, querying records, and managing access, ensuring data integrity and consistency as per ACID properties (referenced in prior discussions).
- **Standardization**: SQL follows standards (e.g., ANSI/ISO SQL), with variations across RDBMS like MySQL, PostgreSQL, and Oracle.
- **Relation to Prior Discussions**:
    - **DML**: SQL’s Data Manipulation Language commands (e.g., `SELECT`, `INSERT`) manipulate entity sets and relationships.
    - **Normalization and Functional Dependency**: SQL commands create and query normalized schemas (e.g., 3NF, BCNF) based on functional dependencies.
    - **ER Model**: SQL translates ER diagrams into tables, with keys and relationships enforced via constraints.
    - **Concurrency Control and Conflict Serializability**: SQL transactions use concurrency control protocols (e.g., locking, MVCC) to ensure serializability.
- **Execution**: SQL commands are processed by the DBMS query engine, leveraging indexing and optimization techniques (referenced in prior indexing discussions) for performance.

## Types of SQL Commands

SQL commands are classified into five main categories based on their functionality:

1. **Data Definition Language (DDL)**:
    
    - **Purpose**: Defines and modifies the structure of database objects (e.g., tables, schemas) in the ER Model.
    - **Commands**:
        - `CREATE`: Defines new database objects (e.g., tables, indexes).
        - `ALTER`: Modifies existing objects (e.g., adds columns, changes constraints).
        - `DROP`: Deletes objects (e.g., tables, databases).
        - `TRUNCATE`: Removes all records from a table without altering its structure.
    - **Characteristics**:
        - Affects the schema, not the data content.
        - Commits automatically in most RDBMS, impacting ACID properties (durability).
    - **Example**:
        
        ```sql
        CREATE TABLE Students (
            StudentID INT PRIMARY KEY,
            Name VARCHAR(50),
            DOB DATE
        );
        ALTER TABLE Students ADD Email VARCHAR(100);
        ```
        
2. **Data Manipulation Language (DML)**:
    
    - **Purpose**: Manipulates data within tables (entity sets), as discussed in prior DML notes.
    - **Commands**:
        - `SELECT`: Retrieves data from tables.
        - `INSERT`: Adds new records to a table.
        - `UPDATE`: Modifies existing records.
        - `DELETE`: Removes records from a table.
    - **Characteristics**:
        - Operates on data within normalized schemas (e.g., 1NF, 2NF).
        - Subject to concurrency control (e.g., locking, MVCC, as discussed in concurrency control protocols).
    - **Example**:
        
        ```sql
        INSERT INTO Students VALUES (101, 'Alice', '2000-01-01', 'alice@example.com');
        SELECT Name, Email FROM Students WHERE StudentID = 101;
        ```
        
3. **Data Control Language (DCL)**:
    
    - **Purpose**: Manages access permissions and security for database objects.
    - **Commands**:
        - `GRANT`: Assigns privileges (e.g., read, write) to users.
        - `REVOKE`: Removes privileges from users.
    - **Characteristics**:
        - Ensures data security and integrity, complementing ACID’s isolation property.
        - Applied to users or roles in RDBMS.
    - **Example**:
        
        ```sql
        GRANT SELECT, INSERT ON Students TO user1;
        REVOKE INSERT ON Students FROM user1;
        ```
        
4. **Transaction Control Language (TCL)**:
    
    - **Purpose**: Manages transactions to ensure ACID compliance, particularly atomicity and durability.
    - **Commands**:
        - `COMMIT`: Saves changes made in a transaction.
        - `ROLLBACK`: Undoes changes in a transaction.
        - `SAVEPOINT`: Sets a point within a transaction for partial rollback.
        - `SET TRANSACTION`: Configures transaction properties (e.g., read-only).
    - **Characteristics**:
        - Works with concurrency control protocols to ensure conflict serializability (referenced in prior discussions).
        - Critical for multi-statement operations (e.g., fund transfers).
    - **Example**:
        
        ```sql
        BEGIN TRANSACTION;
        UPDATE Account SET Balance = Balance - 100 WHERE AccountID = 'A1';
        UPDATE Account SET Balance = Balance + 100 WHERE AccountID = 'A2';
        COMMIT;
        ```
        
5. **Data Query Language (DQL)**:
    
    - **Purpose**: Retrieves data from the database, often considered a subset of DML.
    - **Command**:
        - `SELECT`: Queries data from one or more tables, often using joins, filters, or aggregations.
    - **Characteristics**:
        - Leverages relationships and keys (from ER Model discussions) for joins.
        - Benefits from indexing for performance (referenced in prior indexing discussions).
    - **Example**:
        
        ```sql
        SELECT s.Name, c.CourseName
        FROM Students s
        JOIN Enrollments e ON s.StudentID = e.StudentID
        JOIN Courses c ON e.CourseID = c.CourseID;
        ```
        

## Technical Notes

- **Relation to Prior Discussions**:
    - **ER Model**: DDL commands create tables from entity types and relationships (e.g., `Students`, `Enrollments`).
    - **Normalization**: DDL ensures schemas adhere to normal forms (e.g., 3NF, BCNF), while DML manipulates normalized data.
    - **Functional Dependency**: DML queries respect dependencies (e.g., `StudentID → Name`) to maintain consistency.
    - **Concurrency Control**: TCL commands (`COMMIT`, `ROLLBACK`) work with protocols like two-phase locking or MVCC to ensure conflict serializability.
    - **Denormalization**: DML queries on denormalized tables (e.g., redundant attributes) simplify joins but require careful updates.
    - **Entity Sets**: DML and DQL operate on entity sets (e.g., rows in `Students`), while DDL defines their structure.
- **Performance Considerations**:
    - DQL (`SELECT`) performance improves with indexing and optimized joins.
    - DDL operations (e.g., `ALTER`) may lock tables, impacting concurrency.
    - TCL ensures transaction integrity but requires efficient concurrency control to avoid deadlocks (referenced in prior discussions).
- **Implementation**:
    - Supported by RDBMS like MySQL, PostgreSQL, and Oracle, with slight syntax variations.
    - Tools like MySQL Workbench or pgAdmin assist in writing and testing SQL commands.
    - Refer to _MySQL Reference Manual_ ([https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)) or _PostgreSQL Documentation_ ([https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)) for detailed syntax.

## Practical Project-Related Example

### Project Context: Library Management System

A library database manages books, borrowers, and loans, requiring various SQL commands to define, manipulate, and query data.

- **ER Model Context** (from prior discussions):
    
    - **Entities**: `Book(BookID, Title, Author)`, `Borrower(BorrowerID, Name)`, `Loan(LoanID, BookID, BorrowerID, LoanDate)`.
    - **Relationships**: `Borrows` (many-to-many between `Book` and `Borrower` via `Loan`).
    - **Functional Dependencies**: `BookID → Title, Author`, `BorrowerID → Name`.
- **SQL Commands**:
    
    - **DDL**: Create and modify schema.
        
        ```sql
        CREATE TABLE Book (
            BookID VARCHAR(10) PRIMARY KEY,
            Title VARCHAR(100),
            Author VARCHAR(50)
        );
        CREATE TABLE Borrower (
            BorrowerID VARCHAR(10) PRIMARY KEY,
            Name VARCHAR(50)
        );
        CREATE TABLE Loan (
            LoanID VARCHAR(10) PRIMARY KEY,
            BookID VARCHAR(10),
            BorrowerID VARCHAR(10),
            LoanDate DATE,
            FOREIGN KEY (BookID) REFERENCES Book(BookID),
            FOREIGN KEY (BorrowerID) REFERENCES Borrower(BorrowerID)
        );
        ALTER TABLE Book ADD PublicationYear INT;
        ```
        
    - **DML**: Manipulate data.
        
        ```sql
        INSERT INTO Book VALUES ('B1', 'Database Systems', 'Silberschatz', 2020);
        INSERT INTO Borrower VALUES ('R1', 'Alice');
        INSERT INTO Loan VALUES ('L1', 'B1', 'R1', '2025-06-01');
        UPDATE Book SET PublicationYear = 2021 WHERE BookID = 'B1';
        DELETE FROM Loan WHERE LoanID = 'L1';
        ```
        
    - **DQL**: Query data.
        
        ```sql
        SELECT b.Title, r.Name, l.LoanDate
        FROM Book b
        JOIN Loan l ON b.BookID = l.BookID
        JOIN Borrower r ON l.BorrowerID = r.BorrowerID
        WHERE l.LoanDate >= '2025-01-01';
        ```
        
    - **DCL**: Manage access.
        
        ```sql
        GRANT SELECT, INSERT ON Book TO librarian;
        REVOKE INSERT ON Book FROM librarian;
        ```
        
    - **TCL**: Manage transactions.
        
        ```sql
        BEGIN TRANSACTION;
        INSERT INTO Loan VALUES ('L2', 'B1', 'R1', '2025-06-02');
        UPDATE Book SET PublicationYear = 2022 WHERE BookID = 'B1';
        COMMIT;
        ```
        
- **Use Case**:
    
    - DDL creates normalized tables (3NF, as discussed in normal forms).
    - DML and DQL manage and retrieve loan records, respecting relationships and keys.
    - TCL ensures atomicity for multi-step operations (e.g., loan creation and book update).
    - DCL restricts access to authorized users, supporting data security.
    - Concurrency control (e.g., MVCC in PostgreSQL) ensures conflict serializability for concurrent loan updates.

## Revision Summary

- **Core Concept**: SQL commands (DDL, DML, DCL, TCL, DQL) enable schema definition, data manipulation, access control, transaction management, and querying in RDBMS.
- **Types**:
    - DDL: Defines/modifies schema (`CREATE`, `ALTER`, `DROP`, `TRUNCATE`).
    - DML: Manipulates data (`SELECT`, `INSERT`, `UPDATE`, `DELETE`).
    - DCL: Controls access (`GRANT`, `REVOKE`).
    - TCL: Manages transactions (`COMMIT`, `ROLLBACK`, `SAVEPOINT`).
    - DQL: Queries data (`SELECT`).
- **Relevance to Prior Discussions**: Supports ER Model (schema creation), normalization (table design), functional dependencies (data integrity), concurrency control (transaction isolation), and entity sets (data manipulation).
- **Practical Use**: Applied in systems like library management to manage books, borrowers, and loans efficiently.
- **Further Study**: Consult _Database System Concepts_ by Silberschatz, Korth, and Sudarshan for theoretical foundations, or _MySQL Reference Manual_ ([https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)) and _PostgreSQL Documentation_ ([https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)) for practical syntax.