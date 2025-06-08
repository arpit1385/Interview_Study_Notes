## Definition

Denormalization is the intentional process of reintroducing redundancy into a normalized database schema in a Relational Database Management System (RDBMS) to optimize query performance, typically at the expense of data integrity or increased storage requirements. It reverses aspects of normalization (previously discussed) by combining tables, adding redundant attributes, or relaxing normal form constraints to enhance read efficiency.

## Characteristics

- **Purpose**: Improves performance for read-heavy operations (e.g., reporting, analytics) by reducing the need for complex joins or computations, as opposed to normalization, which prioritizes data integrity and eliminates redundancy.
- **Trade-offs**: Increases risk of data anomalies (e.g., insert, update, or delete anomalies) and storage costs but simplifies queries and improves response times.
- **Context**: Commonly applied in data warehouses, reporting systems, or applications where read performance is critical, unlike transactional systems where normalization ensures ACID compliance (referenced in prior discussions).
- **Relation to Normalization**: Denormalization is applied to a normalized schema (e.g., in 3NF or BCNF) to balance performance and integrity based on specific use cases.

## Types of Denormalization

Denormalization can be achieved through several techniques, each addressing specific performance needs:

1. **Table Combination**:
    
    - Merges normalized tables to eliminate joins, reducing query complexity.
    - Example: Combining `Orders` and `OrderItems` tables (from prior normalization discussion) into a single table with redundant data like `CustomerName`.
2. **Redundant Attribute Addition**:
    
    - Adds derived or frequently accessed attributes to tables to avoid calculations or joins.
    - Example: Storing `TotalOrderValue` in an `Orders` table, even though it can be computed from `OrderItems`.
3. **Reintroduction of Multi-Valued Attributes**:
    
    - Stores non-atomic data (violating 1NF) to simplify data retrieval.
    - Example: Including a comma-separated list of `ProductIDs` in an `Orders` table instead of a separate `OrderItems` table.
4. **Pre-Aggregation**:
    
    - Stores pre-computed aggregates (e.g., sums, averages) to avoid runtime calculations.
    - Example: Maintaining a `TotalSales` column in a `Customers` table for quick reporting.
5. **Star Schema Creation (Data Warehousing)**:
    
    - Organizes data into a central fact table with denormalized dimension tables to optimize analytical queries.
    - Example: A fact table for `Sales` with denormalized `Customer` and `Product` details in dimension tables.

## Conversion Methods (Denormalization Process)

Denormalization involves modifying a normalized schema to introduce controlled redundancy. The process includes:

1. **Analyze Query Patterns**:
    
    - Identify frequently executed queries (e.g., `SELECT` statements from DML discussions) that involve multiple joins or complex calculations.
    - Example: A report joining `Orders`, `Customers`, and `OrderItems` repeatedly may benefit from denormalization.
2. **Select Denormalization Type**:
    
    - Choose a technique (e.g., table combination, redundant attributes) based on performance needs and acceptable trade-offs.
    - Example: Add `CustomerName` to `OrderItems` to eliminate joins with `Customers`.
3. **Modify Schema**:
    
    - Alter tables using DDL commands (e.g., `ALTER TABLE`) to add columns, merge tables, or store non-atomic data.
    - Example: `ALTER TABLE OrderItems ADD CustomerName VARCHAR(100);`.
4. **Update Data**:
    
    - Populate redundant or derived attributes using DML commands (e.g., `UPDATE` or `INSERT`) or triggers to maintain consistency.
    - Example: `UPDATE OrderItems SET CustomerName = (SELECT CustomerName FROM Customers WHERE Customers.CustomerID = OrderItems.CustomerID);`.
5. **Implement Maintenance Mechanisms**:
    
    - Use triggers, stored procedures, or application logic to synchronize redundant data and prevent anomalies.
    - Example: A trigger on `Customers` to update `CustomerName` in `OrderItems` when `CustomerName` changes.
6. **Optimize with Indexing**:
    
    - Apply indexing (referenced in prior discussions) to denormalized tables to further enhance query performance.
    - Example: Create an index on `CustomerName` in a denormalized `OrderItems` table.

## Reverse Conversion (Renormalization)

Renormalization involves reverting a denormalized schema to a normalized form to restore data integrity or reduce redundancy:

- **Process**:
    - Identify redundant or non-atomic attributes and decompose tables to meet normal form requirements (e.g., 1NF, 2NF, 3NF, as discussed in prior normalization notes).
    - Example: Split a denormalized `Orders` table with `CustomerName` and `ProductIDs` into `Orders`, `Customers`, and `OrderItems`.
- **Steps**:
    1. Remove redundant attributes using DDL (e.g., `ALTER TABLE Orders DROP COLUMN CustomerName;`).
    2. Create separate tables for related data, ensuring proper keys and relationships.
    3. Migrate data using DML to populate normalized tables.
    4. Re-establish constraints (e.g., foreign keys) to ensure integrity.
- **Use Case**: Applied when data integrity becomes critical (e.g., transitioning from a reporting system to a transactional system).

## Technical Notes

- **Relation to Prior Discussions**:
    - **Normalization**: Denormalization reverses normalization by reintroducing redundancy, contrasting with the goal of eliminating anomalies via normal forms (1NF, 2NF, 3NF, BCNF).
    - **DML**: Denormalized schemas simplify `SELECT` queries but may complicate `UPDATE`, `INSERT`, and `DELETE` operations due to redundancy.
    - **ACID Properties**: Denormalization risks consistency (e.g., update anomalies), requiring careful management to maintain ACID compliance.
    - **Indexing and Sharding**: Denormalized tables benefit from indexing for faster reads and may align with sharding for horizontal scaling (referenced in prior scaling discussions).
    - **Keys and Relationships**: Denormalization may weaken referential integrity, requiring alternative mechanisms like triggers to enforce relationships.
- **Performance Considerations**: Denormalization reduces join overhead but increases storage and maintenance costs. It is ideal for read-heavy systems like data warehouses.
- **Tools**: Modern RDBMS (e.g., MySQL, PostgreSQL) support denormalization through flexible schema modifications. Refer to _MySQL Reference Manual_ ([https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)) or _PostgreSQL Documentation_ ([https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)) for DDL/DML syntax.

## Practical Real-Life Example

**Scenario**: An e-commerce platform maintains a normalized database with tables `Orders` (OrderID, CustomerID, OrderDate), `Customers` (CustomerID, CustomerName, Address), and `OrderItems` (OrderID, ProductID, Quantity). Frequent sales reports require joining these tables to display order details, customer names, and product information, leading to performance bottlenecks.

**Denormalization Applied**:

- **Type**: Table combination and redundant attribute addition.
- **Conversion**:
    - Combine `Orders` and `OrderItems` into a denormalized `OrderDetails` table with columns: `OrderID`, `CustomerID`, `CustomerName`, `OrderDate`, `ProductID`, `Quantity`.
    - Add a derived column `TotalPrice` (calculated as `Quantity * UnitPrice`) to avoid runtime calculations.
    - Use DML: `INSERT INTO OrderDetails SELECT o.OrderID, o.CustomerID, c.CustomerName, o.OrderDate, oi.ProductID, oi.Quantity, (oi.Quantity * p.UnitPrice) AS TotalPrice FROM Orders o JOIN Customers c ON o.CustomerID = c.CustomerID JOIN OrderItems oi ON o.OrderID = oi.OrderID JOIN Products p ON oi.ProductID = p.ProductID;`.
    - Create an index on `CustomerName` for faster report queries.
- **Maintenance**: Add a trigger to update `CustomerName` in `OrderDetails` when `Customers.CustomerName` changes.
- **Outcome**: Sales reports run faster due to fewer joins, but updates to `CustomerName` require additional synchronization.

**Renormalization (if needed)**:

- Split `OrderDetails` back into `Orders`, `Customers`, and `OrderItems`.
- Remove `CustomerName` and `TotalPrice` from `OrderDetails`.
- Re-establish foreign key constraints to ensure referential integrity.

## Revision Summary

- **Core Concept**: Denormalization introduces controlled redundancy to optimize read performance, reversing normalization principles.
- **Types**: Table combination, redundant attributes, multi-valued attributes, pre-aggregation, and star schema.
- **Conversion**: Analyze queries, modify schema, update data, and implement maintenance mechanisms.
- **Renormalization**: Reverts to normalized forms to restore integrity.
- **Relevance to Prior Discussions**: Contrasts with normalization, impacts DML operations, and aligns with indexing and sharding for performance.
- **Practical Use**: Common in data warehouses or reporting systems, as shown in the e-commerce example.
- **Further Study**: Consult _Database System Concepts_ by Silberschatz, Korth, and Sudarshan for theoretical insights, or MySQL/PostgreSQL documentation for practical implementation.