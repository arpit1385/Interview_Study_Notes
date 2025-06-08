# Conflict Serializability in DBMS

## Definition

Conflict Serializability is a property of a schedule (sequence of operations) in a Database Management System (DBMS) that ensures a concurrent execution of multiple transactions produces the same result as some serial (sequential) execution of those transactions. It is a critical concept in transaction management, ensuring that concurrent transactions maintain the consistency and isolation properties of ACID (Atomicity, Consistency, Isolation, Durability), as referenced in prior discussions.

## Characteristics

- **Purpose**: Guarantees that a concurrent schedule preserves database consistency by being equivalent to a serial schedule, preventing conflicts that could lead to incorrect data states.
- **Conflict Operations**: Two operations from different transactions conflict if:
    - They operate on the same data item.
    - At least one operation is a write (e.g., `UPDATE`, `INSERT`, `DELETE` from DML discussions).
    - Examples of conflicting operations: Transaction ( T_1 ) writes to data item ( A ), and Transaction ( T_2 ) reads or writes to ( A ).
- **Non-Conflicting Operations**: Operations that do not conflict (e.g., both are reads or access different data items) can be reordered without affecting the outcome.
- **Serial Schedule**: A schedule where transactions execute one after another without interleaving, ensuring no conflicts.
- **Conflict Equivalent**: Two schedules are conflict equivalent if they involve the same transactions and produce the same final database state by reordering non-conflicting operations.
- **Test for Conflict Serializability**: Uses a precedence graph (conflict graph) to determine if a schedule is conflict serializable.

## Technical Notes

- **Precedence Graph**:
    - **Construction**: Create a directed graph where nodes represent transactions (( T_1, T_2, \ldots )), and an edge from ( T_i \rightarrow T_j ) exists if an operation in ( T_i ) precedes and conflicts with an operation in ( T_j ).
    - **Condition**: A schedule is conflict serializable if its precedence graph is acyclic (no cycles). An acyclic graph implies a topological order equivalent to a serial schedule.
- **Relation to ACID Properties**:
    - **Isolation**: Conflict serializability ensures that concurrent transactions appear to execute in isolation, as if in a serial order.
    - **Consistency**: Maintains functional dependencies and constraints (referenced in prior discussions) by preventing conflicting operations from producing inconsistent states.
- **Relation to Prior Discussions**:
    - **DML**: Conflict serializability governs the execution of DML operations (`SELECT`, `UPDATE`, etc.) in concurrent transactions to avoid conflicts.
    - **Normalization and Functional Dependency**: Ensures that normalized schemas (e.g., 3NF) remain consistent under concurrent updates, respecting functional dependencies.
    - **ER Model and Keys**: Relationships and keys defined in the ER Model guide which data items are accessed, influencing potential conflicts.
    - **Concurrency Control**: Techniques like locking, timestamp ordering, or multiversion concurrency control (MVCC) enforce conflict serializability, as used in modern RDBMS like MySQL and PostgreSQL.
- **Practical Implementation**:
    - DBMS schedulers use conflict serializability to validate schedules, rejecting those that are not serializable.
    - Tools like MySQL’s InnoDB or PostgreSQL’s MVCC ensure serializability by managing locks or versioned data.
- **Limitations**:
    - Ensuring conflict serializability may reduce concurrency (e.g., via locking), impacting performance.
    - Not all serializable schedules are conflict serializable; view serializability is a broader concept but less commonly enforced due to complexity.

## Example

Consider two transactions ( T_1 ) and ( T_2 ) operating on data items ( A ) and ( B ):

- ( T_1 ): `Read(A)`, `A = A + 10`, `Write(A)`, `Read(B)`, `B = B + 20`, `Write(B)`
- ( T_2 ): `Read(A)`, `A = A * 2`, `Write(A)`

**Concurrent Schedule**:

```
S: Read1(A), Write1(A), Read2(A), Write2(A), Read1(B), Write1(B)
```

- **Conflicts**:
    - ( Write1(A) \rightarrow Read2(A) ): ( T_1 ) writes ( A ), ( T_2 ) reads ( A ).
    - ( Write1(A) \rightarrow Write2(A) ): Both write to ( A ).
- **Precedence Graph**:
    - Nodes: ( T_1 ), ( T_2 ).
    - Edges: ( T_1 \rightarrow T_2 ) (due to conflicts on ( A )).
    - Graph is acyclic, so the schedule is conflict serializable, equivalent to serial order ( T_1 \rightarrow T_2 ).

**Non-Serializable Schedule**:

```
S': Read1(A), Read2(A), Write2(A), Write1(A), Read1(B), Write1(B)
```

- **Conflicts**: ( Read1(A) \rightarrow Write2(A) ), ( Write2(A) \rightarrow Write1(A) ).
- **Precedence Graph**: Edges ( T_1 \rightarrow T_2 ) and ( T_2 \rightarrow T_1 ), forming a cycle.
- **Conclusion**: ( S' ) is not conflict serializable due to the cycle.

## Practical Project-Related Example

**Scenario**: An online banking system manages accounts and transactions, with multiple users transferring funds concurrently.

- **ER Model Context** (from prior discussions):
    - **Entities**: `Account(AccountID, Balance)`, `Transaction(TransactionID, FromAccountID, ToAccountID, Amount)`.
    - **Relationships**: `Transfers` (links `Account` to `Transaction`).
    - **Functional Dependencies**: `AccountID → Balance`.
- **Transactions**:
    - ( T_1 ): Transfer $100 from Account A to Account B.
        - `Read(A.Balance)`, `A.Balance = A.Balance - 100`, `Write(A.Balance)`, `Read(B.Balance)`, `B.Balance = B.Balance + 100`, `Write(B.Balance)`.
    - ( T_2 ): Transfer $50 from Account B to Account C.
        - `Read(B.Balance)`, `B.Balance = B.Balance - 50`, `Write(B.Balance)`, `Read(C.Balance)`, `C.Balance = C.Balance + 50`, `Write(C.Balance)`.
- **Concurrent Schedule**:
    
    ```
    S: Read1(A.Balance), Write1(A.Balance), Read1(B.Balance), Read2(B.Balance), Write2(B.Balance), Write1(B.Balance), Read2(C.Balance), Write2(C.Balance)
    ```
    
- **Precedence Graph**:
    - Conflicts: ( Read1(B.Balance) \rightarrow Write2(B.Balance) ), ( Write2(B.Balance) \rightarrow Write1(B.Balance) ).
    - Edges: ( T_1 \rightarrow T_2 ), ( T_2 \rightarrow T_1 ).
    - Cycle exists, so the schedule is not conflict serializable.
- **Resolution**: Use locking (e.g., two-phase locking) to enforce a serial order (e.g., ( T_1 \rightarrow T_2 )) or adjust the schedule to eliminate conflicts.
- **Outcome**: Ensures account balances remain consistent (e.g., no double-spending), aligning with ACID properties.

## Revision Summary

- **Core Concept**: Conflict serializability ensures a concurrent transaction schedule is equivalent to a serial schedule, preserving consistency and isolation.
- **Mechanism**: Uses precedence graphs to test for acyclicity, identifying conflict serializable schedules.
- **Relevance to Prior Discussions**: Builds on ACID properties (isolation, consistency), DML (operation scheduling), normalization (maintaining functional dependencies), and ER Model (defining data items).
- **Practical Use**: Critical in transaction-heavy systems like banking to ensure data integrity under concurrency.
- **Further Study**: Consult _Database System Concepts_ by Silberschatz, Korth, and Sudarshan for theoretical details, or _MySQL Reference Manual_ ([https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)) and _PostgreSQL Documentation_ ([https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)) for concurrency control mechanisms.