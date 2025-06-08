# Nested Queries in SQL

## Definition

A nested query, also known as a subquery, is an SQL query embedded within another SQL query, typically enclosed in parentheses, used to retrieve data that serves as input for the outer (main) query. Nested queries are primarily part of Data Query Language (DQL) and Data Manipulation Language (DML) operations (referenced in prior SQL commands discussion) and are executed within the context of a larger query to filter, compute, or compare data. They operate on entity sets and relationships defined in the Entity-Relationship (ER) Model, leveraging keys and functional dependencies (as discussed previously).

## Characteristics

- **Purpose**: Enables complex queries by breaking them into smaller, manageable subqueries that return intermediate results used by the outer query.
- **Execution**: The subquery is evaluated first, and its result is passed to the outer query. The DBMS query optimizer (leveraging indexing, as discussed previously) determines the execution order for efficiency.
- **Placement**: Subqueries can appear in clauses like `SELECT`, `WHERE`, `FROM`, or `HAVING`.
- **Relation to Prior Discussions**:
    - **DML/DQL**: Nested queries are a subset of `SELECT` statements, often used to filter data based on relationships or keys.
    - **ER Model**: Subqueries operate on entity sets (e.g., tables like `Students`, `Courses`) and relationships (e.g., `Enrollments`).
    - **Normalization**: Subqueries respect normalized schemas (e.g., 3NF, BCNF) to maintain data integrity.
    - **Concurrency Control**: Nested queries in transactions must adhere to conflict serializability (as discussed in concurrency control protocols) to ensure consistent results.
    - **ACID Properties**: Subqueries support consistency and isolation in multi-user environments.

## Types of Nested Queries

Nested queries are classified based on their structure, return type, and usage. Below are the primary types, each with a practical example.

### 1. Single-Row Subquery

- **Definition**: A subquery that returns a single row with one column, often used with comparison operators (e.g., `=`, `>`, `<`).
- **Characteristics**:
    - Returns a scalar value.
    - Commonly used in `WHERE` or `HAVING` clauses to filter results.
    - Must return exactly one row; otherwise, an error occurs.
- **Practical Example**: In a university database (referenced in prior discussions), find students whose age is greater than the average age of all students.
    - **Schema**:
        
        ```sql
        CREATE TABLE Students (
            StudentID INT PRIMARY KEY,
            Name VARCHAR(50),
            DOB DATE
        );
        INSERT INTO Students VALUES (101, 'Alice', '2000-01-01'), (102, 'Bob', '1998-05-15'), (103, 'Charlie', '2002-03-10');
        ```
        
    - **SQL Query**:
        
        ```sql
        SELECT Name
        FROM Students
        WHERE DATEDIFF(YEAR, DOB, GETDATE()) > (
            SELECT AVG(DATEDIFF(YEAR, DOB, GETDATE()))
            FROM Students
        );
        ```
        
    - **Explanation**: The subquery `(SELECT AVG(DATEDIFF(YEAR, DOB, GETDATE())) FROM Students)` calculates the average age (e.g., 25 years). The outer query retrieves names of students older than this average (e.g., Bob, born in 1998).

### 2. Multi-Row Subquery

- **Definition**: A subquery that returns multiple rows, typically with one column, used with operators like `IN`, `ANY`, `ALL`.
- **Characteristics**:
    - Returns a set of values for comparison.
    - Used to filter rows based on a list or range of values.
- **Practical Example**: Find students enrolled in a specific department’s courses.
    - **Schema** (from prior discussions):
        
        ```sql
        CREATE TABLE Courses (
            CourseID VARCHAR(10) PRIMARY KEY,
            CourseName VARCHAR(50),
            Department VARCHAR(50)
        );
        CREATE TABLE Enrollments (
            StudentID INT,
            CourseID VARCHAR(10),
            PRIMARY KEY (StudentID, CourseID),
            FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
            FOREIGN KEY (CourseID) REFERENCES Courses(CourseID)
        );
        INSERT INTO Courses VALUES ('CS101', 'Database', 'Computer Science'), ('MA101', 'Calculus', 'Mathematics');
        INSERT INTO Enrollments VALUES (101, 'CS101'), (102, 'CS101'), (103, 'MA101');
        ```
        
    - **SQL Query**:
        
        ```sql
        SELECT Name
        FROM Students
        WHERE StudentID IN (
            SELECT StudentID
            FROM Enrollments
            WHERE CourseID IN (
                SELECT CourseID
                FROM Courses
                WHERE Department = 'Computer Science'
            )
        );
        ```
        
    - **Explanation**: The innermost subquery `(SELECT CourseID FROM Courses WHERE Department = 'Computer Science')` returns `CS101`. The middle subquery `(SELECT StudentID FROM Enrollments WHERE CourseID IN (...))` returns `StudentID`s 101 and 102. The outer query retrieves names (Alice, Bob).

### 3. Correlated Subquery

- **Definition**: A subquery that references columns from the outer query, executed repeatedly for each row of the outer query.
- **Characteristics**:
    - Tightly coupled with the outer query, often used for row-by-row comparisons.
    - Can be less efficient due to repeated execution but powerful for complex conditions.
- **Practical Example**: Find students who are enrolled in at least one course taught by a specific instructor.
    - **Schema**:
        
        ```sql
        CREATE TABLE Instructors (
            InstructorID INT PRIMARY KEY,
            Name VARCHAR(50)
        );
        ALTER TABLE Courses ADD InstructorID INT,
        ADD FOREIGN KEY (InstructorID) REFERENCES Instructors(InstructorID);
        INSERT INTO Instructors VALUES (201, 'Prof. Smith');
        UPDATE Courses SET InstructorID = 201 WHERE CourseID = 'CS101';
        ```
        
    - **SQL Query**:
        
        ```sql
        SELECT Name
        FROM Students s
        WHERE EXISTS (
            SELECT 1
            FROM Enrollments e
            JOIN Courses c ON e.CourseID = c.CourseID
            WHERE e.StudentID = s.StudentID
            AND c.InstructorID = 201
        );
        ```
        
    - **Explanation**: For each `Student` row, the subquery checks if there exists an `Enrollment` for that `StudentID` linked to a course taught by `InstructorID = 201`. Returns Alice and Bob, enrolled in `CS101`.

### 4. Nested Query in FROM Clause (Derived Table)

- **Definition**: A subquery used in the `FROM` clause, treated as a temporary table (derived table) for the outer query.
- **Characteristics**:
    - Useful for aggregating or transforming data before joining or filtering.
    - Requires an alias for the derived table.
- **Practical Example**: Find the total number of enrollments per department.
    - **SQL Query**:
        
        ```sql
        SELECT Department, COUNT(*) AS EnrollmentCount
        FROM (
            SELECT c.Department
            FROM Enrollments e
            JOIN Courses c ON e.CourseID = c.CourseID
        ) AS DerivedTable
        GROUP BY Department;
        ```
        
    - **Explanation**: The subquery `(SELECT c.Department FROM Enrollments e JOIN Courses c ON e.CourseID = c.CourseID)` creates a derived table with department names for all enrollments. The outer query counts enrollments per department (e.g., Computer Science: 2, Mathematics: 1).

### 5. Nested Query in SELECT Clause

- **Definition**: A subquery embedded in the `SELECT` clause to compute a value for each row of the outer query.
- **Characteristics**:
    - Typically returns a single value per row.
    - Useful for computed columns or comparisons.
- **Practical Example**: Display each student’s name and the number of courses they are enrolled in.
    - **SQL Query**:
        
        ```sql
        SELECT Name,
               (SELECT COUNT(*)
                FROM Enrollments e
                WHERE e.StudentID = s.StudentID) AS CourseCount
        FROM Students s;
        ```
        
    - **Explanation**: For each student, the subquery `(SELECT COUNT(*) FROM Enrollments e WHERE e.StudentID = s.StudentID)` counts their enrollments. Returns, e.g., Alice: 2, Bob: 1, Charlie: 1.

## Technical Notes

- **Relation to Prior Discussions**:
    - **DML/DQL**: Nested queries are primarily `SELECT` statements (DQL), used within DML operations to filter or compute data.
    - **ER Model**: Operate on entity sets (e.g., `Students`, `Courses`) and relationships (e.g., `Enrollments`), as defined in the ER Model.
    - **Normalization**: Subqueries work with normalized schemas (e.g., 3NF), respecting functional dependencies (e.g., `StudentID → Name`).
    - **Concurrency Control**: In concurrent environments, subqueries must be managed by protocols like MVCC or two-phase locking (referenced in concurrency control protocols) to ensure conflict serializability.
    - **Indexing**: Indexes on columns like `StudentID` or `CourseID` (as discussed in indexing) improve subquery performance.
    - **ACID Properties**: Nested queries maintain consistency and isolation in transactions, aligning with ACID principles.
- **Performance Considerations**:
    - Correlated subqueries can be slow due to row-by-row execution; joins may be more efficient.
    - Single-row subqueries require careful design to avoid errors (e.g., multiple rows returned).
    - Indexing and query optimization (e.g., MySQL’s EXPLAIN) enhance performance.
- **Implementation**: Supported by RDBMS like MySQL, PostgreSQL, and Oracle, with minor syntax variations. Refer to _MySQL Reference Manual_ ([https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)) or _PostgreSQL Documentation_ ([https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)) for details.

## Revision Summary

- **Core Concept**: Nested queries (subqueries) are SQL queries embedded within another query, used to filter, compute, or transform data.
- **Types**:
    - Single-Row: Returns one value (e.g., compare with average age).
    - Multi-Row: Returns multiple values (e.g., students in a department).
    - Correlated: References outer query (e.g., students with specific instructor).
    - FROM Clause: Derived table for aggregation (e.g., enrollment counts).
    - SELECT Clause: Computes per-row values (e.g., course counts).
- **Relevance to Prior Discussions**: Builds on DML/DQL, ER Model, normalization, functional dependencies, concurrency control, and indexing.
- **Practical Use**: Enables complex queries in systems like university databases, leveraging relationships and keys.
- **Further Study**: Consult _Database System Concepts_ by Silberschatz, Korth, and Sudarshan for theoretical foundations, or MySQL/PostgreSQL documentation for practical syntax.