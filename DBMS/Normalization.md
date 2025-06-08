## Definition

Normalization is a systematic process in a Relational Database Management System (RDBMS) to design databases by organizing data into tables to eliminate redundancy and ensure data integrity. It involves decomposing a larger table into smaller, well-structured tables that adhere to specific rules called normal forms, which are based on functional dependencies and keys (as referenced in prior discussions on RDBMS and keys).

## Characteristics

- **Purpose**: Reduces data redundancy, prevents anomalies during data manipulation (insert, update, delete as covered in DML), and ensures logical data consistency.
- **Basis**: Relies on functional dependencies, where one attribute uniquely determines another, and primary/foreign keys to maintain relationships.
- **Normal Forms**: Progressive levels (1NF, 2NF, 3NF, BCNF, etc.) with increasing constraints to achieve optimal structure.
- **Trade-offs**: Enhances data integrity but may increase query complexity, potentially requiring joins (referenced in prior discussions on relationships).

## Types of Normalization

Normalization progresses through a series of normal forms, each addressing specific anomalies:

1. **First Normal Form (1NF)**:
    
    - Ensures all attributes are atomic (indivisible) and each record is unique.
    - Requires a primary key and eliminates repeating groups or arrays.
    - Example: A table with a column storing multiple phone numbers is split into separate rows or a related table.
2. **Second Normal Form (2NF)**:
    
    - Must be in 1NF.
    - Eliminates partial dependencies, where non-key attributes depend on only part of a composite primary key.
    - Example: In a table with `OrderID` and `ProductID` as a composite key, attributes like `CustomerName` (dependent only on `OrderID`) are moved to a separate table.
3. **Third Normal Form (3NF)**:
    
    - Must be in 2NF.
    - Removes transitive dependencies, where non-key attributes depend on other non-key attributes.
    - Example: If `EmployeeID` determines `Department`, and `Department` determines `DepartmentLocation`, `DepartmentLocation` is moved to a separate table.
4. **Boyce-Codd Normal Form (BCNF)**:
    
    - A stricter version of 3NF.
    - Ensures every determinant (attribute determining another) is a candidate key.
    - Example: In a table with multiple candidate keys, any dependency violating this rule is resolved by decomposition.
5. **Higher Normal Forms (4NF, 5NF)**:
    
    - **4NF**: Addresses multi-valued dependencies, ensuring no unrelated multi-valued attributes exist in the same table.
    - **5NF** (Projection-Join Normal Form): Ensures tables cannot be decomposed into smaller tables without losing data, addressing join dependencies.
    - These are less common in practice but critical for complex databases.

## Conversion Methods (Normalization Process)

The process of normalizing a database involves iterative steps to achieve the desired normal form:

1. **Identify Functional Dependencies**:
    
    - Determine how attributes relate (e.g., `EmployeeID → Name, Department`).
    - Use prior knowledge of keys and relationships to map dependencies.
2. **Apply 1NF**:
    
    - Ensure atomic attributes and unique records.
    - Example: Convert a table with a column `Skills` containing "SQL, Python" into separate rows or a related table.
3. **Apply 2NF**:
    
    - Identify partial dependencies in tables with composite keys.
    - Decompose tables to separate attributes dependent on part of the key.
    - Example: Split an `OrderDetails` table into `Orders` (OrderID, CustomerName) and `OrderItems` (OrderID, ProductID, Quantity).
4. **Apply 3NF**:
    
    - Identify transitive dependencies.
    - Create new tables for attributes dependent on non-key attributes.
    - Example: Move `DepartmentLocation` to a `Departments` table if it depends on `Department`.
5. **Apply BCNF (if needed)**:
    
    - Check for non-candidate key determinants.
    - Decompose tables to ensure all determinants are candidate keys.
    - Example: If `InstructorID → Course` and `Course → Department`, but `Course` is not a candidate key, split into separate tables.
6. **Higher Normal Forms**:
    
    - For 4NF, eliminate multi-valued dependencies by creating separate tables for each multi-valued attribute.
    - For 5NF, ensure no join dependencies exist by verifying lossless decomposition.

## Reverse Conversion (Denormalization)

Denormalization is the intentional reversal of normalization to improve query performance by reintroducing redundancy or combining tables, often at the cost of data integrity.

- **Purpose**: Enhances read performance for specific use cases (e.g., reporting, analytics) by reducing the need for complex joins or improving indexing efficiency (referenced in prior discussions on indexing and query optimization).
- **Methods**:
    - **Combine Tables**: Merge normalized tables to reduce joins. Example: Combine `Orders` and `OrderItems` into a single table with redundant `CustomerName` data.
    - **Add Redundant Attributes**: Store derived or frequently accessed data, such as `TotalPrice` in an `Orders` table, despite it being calculable from `OrderItems`.
    - **Reintroduce Multi-Valued Attributes**: Store non-atomic data, like a comma-separated `Skills` list, to avoid additional tables.
- **Considerations**:
    - Increases risk of anomalies (e.g., update anomalies due to redundant data).
    - Requires careful management, often using triggers or application logic to maintain consistency.
    - Common in data warehouses where read-heavy operations dominate.
- **Example**: A normalized `Employees` and `Departments` table might be denormalized by adding `DepartmentName` to `Employees` to avoid joins in frequent queries.

## Technical Notes

- **Relation to Prior Discussions**:
    - **Keys and Relationships**: Normalization relies on primary and foreign keys to maintain referential integrity during decomposition.
    - **DML**: Post-normalization, DML operations (e.g., `SELECT` with joins) may become more complex, necessitating indexing for performance.
    - **ACID Properties**: Normalization supports consistency and isolation by reducing anomalies, aligning with transaction management.
    - **Sharding and Scaling**: Highly normalized databases may require sharding for horizontal scaling, as discussed previously, to distribute data across nodes.
- **Performance Trade-offs**: Normalization reduces storage needs but may increase query complexity. Denormalization optimizes for read-heavy workloads but risks inconsistency.
- **Tools and Standards**: Modern RDBMS like MySQL and PostgreSQL support normalization through schema design tools. Refer to _MySQL Reference Manual_ ([https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)) or _PostgreSQL Documentation_ ([https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)) for implementation details.

## Example

**Unnormalized Table**:

|OrderID|CustomerName|ProductIDs|Quantities|
|---|---|---|---|
|1|John Doe|P1, P2|2, 3|

**1NF Conversion** (atomic attributes, unique records):

|OrderID|CustomerName|ProductID|Quantity|
|---|---|---|---|
|1|John Doe|P1|2|
|1|John Doe|P2|3|

**2NF Conversion** (remove partial dependency):

- `Orders`: | OrderID | CustomerName |
- `OrderItems`: | OrderID | ProductID | Quantity |

**3NF Conversion** (if `CustomerName → Address` exists, move to separate table):

- `Customers`: | CustomerID | CustomerName | Address |
- `Orders`: | OrderID | CustomerID |
- `OrderItems`: | OrderID | ProductID | Quantity |

**Denormalization Example**:  
Combine `Orders` and `OrderItems` to include `CustomerName` in `OrderItems` for faster queries, accepting redundancy.

## Revision Summary

- **Core Concept**: Normalization organizes data into tables to eliminate redundancy and anomalies, guided by normal forms (1NF, 2NF, 3NF, BCNF, 4NF, 5NF).
- **Types**: Progressive normal forms address atomicity, partial dependencies, transitive dependencies, and more complex dependencies.
- **Conversion**: Iterative decomposition based on functional dependencies and keys.
- **Denormalization**: Reintroduces redundancy for performance, balancing integrity and efficiency.
- **Relevance to Prior Discussions**: Builds on keys, relationships, DML, ACID properties, and indexing for practical implementation.
- **Further Study**: Consult _Database System Concepts_ by Silberschatz, Korth, and Sudarshan for theoretical depth, or MySQL/PostgreSQL manuals for practical guidance.