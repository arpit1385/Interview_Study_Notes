# Normal Forms in DBMS

## Definition

Normal Forms are a set of rules applied during the normalization process in a Relational Database Management System (RDBMS) to design databases that eliminate redundancy and prevent data anomalies during Data Manipulation Language (DML) operations (e.g., `INSERT`, `UPDATE`, `DELETE`, as discussed previously). Each normal form builds on the previous one, addressing specific types of dependencies identified through functional dependencies (referenced in prior discussions) to ensure data integrity and consistency.

## Characteristics

- **Purpose**: Organizes data into tables to minimize redundancy, ensure data integrity, and support the consistency aspect of ACID properties (as discussed in prior notes on ACID and conflict serializability).
- **Basis**: Relies on functional dependencies, keys, and relationships (referenced in ER Model and functional dependency discussions) to decompose tables systematically.
- **Progressive Nature**: Each normal form imposes stricter constraints, from First Normal Form (1NF) to higher forms like Fifth Normal Form (5NF).
- **Relation to Prior Discussions**:
    - **Functional Dependency**: Identifies dependencies (e.g., ( X \rightarrow Y )) to guide normalization.
    - **ER Model**: Provides the conceptual framework for entities and relationships, which normalization refines into relational schemas.
    - **DML**: Ensures DML operations execute without anomalies in normalized schemas.
    - **Denormalization**: Reverses normalization for performance, as discussed previously.

## Types of Normal Forms

The following are the primary normal forms, each addressing specific anomalies or dependencies:

1. **First Normal Form (1NF)**:
    
    - **Rule**: All attributes must be atomic (indivisible), and each record must be unique (typically enforced by a primary key).
    - **Purpose**: Eliminates repeating groups or multi-valued attributes.
    - **Example Issue**: A table with a column storing multiple values (e.g., `Skills` as "SQL, Python") violates 1NF.
    - **Solution**: Split multi-valued attributes into separate rows or tables.
2. **Second Normal Form (2NF)**:
    
    - **Rule**: Must be in 1NF, and no non-key attribute should be partially dependent on a composite primary key (i.e., dependent on only part of the key).
    - **Purpose**: Removes partial dependencies to prevent update anomalies.
    - **Example Issue**: In a table with a composite key (`OrderID`, `ProductID`), `CustomerName` depending only on `OrderID` violates 2NF.
    - **Solution**: Decompose into tables where non-key attributes depend fully on the primary key.
3. **Third Normal Form (3NF)**:
    
    - **Rule**: Must be in 2NF, and no non-key attribute should be transitively dependent on the primary key (i.e., dependent on another non-key attribute).
    - **Purpose**: Eliminates transitive dependencies to further reduce redundancy.
    - **Example Issue**: If `EmployeeID → Department` and `Department → DepartmentLocation`, `DepartmentLocation` is transitively dependent, violating 3NF.
    - **Solution**: Move transitively dependent attributes to a separate table.
4. **Boyce-Codd Normal Form (BCNF)**:
    
    - **Rule**: Must be in 3NF, and for every functional dependency ( X \rightarrow Y ), ( X ) must be a superkey (a set of attributes that uniquely identifies a tuple).
    - **Purpose**: Addresses cases where non-key attributes determine other attributes, ensuring stricter dependency rules.
    - **Example Issue**: If `InstructorID → Course` and `Course → Department`, but `Course` is not a superkey, it violates BCNF.
    - **Solution**: Decompose to ensure all determinants are superkeys.
5. **Fourth Normal Form (4NF)**:
    
    - **Rule**: Must be in BCNF, and no multi-valued dependencies exist unless they are trivial (i.e., no independent multi-valued attributes).
    - **Purpose**: Eliminates multi-valued dependencies not related to the key.
    - **Example Issue**: A table with `StudentID`, `Course`, and `Hobby` where `Course` and `Hobby` are independent multi-valued attributes violates 4NF.
    - **Solution**: Split into separate tables for each multi-valued dependency.
6. **Fifth Normal Form (5NF) or Project-Join Normal Form (PJNF)**:
    
    - **Rule**: Must be in 4NF, and the table cannot be decomposed into smaller tables without loss of data (no join dependencies).
    - **Purpose**: Ensures lossless decomposition when joining tables back together.
    - **Example Issue**: A table with `Supplier`, `Product`, and `Project` where join dependencies exist (e.g., reconstructing the table requires multiple joins).
    - **Solution**: Decompose into smaller tables that preserve all data upon joining.

## Practical Example Using SQL

Consider a university database with an unnormalized table `StudentRecords` containing student, course, and instructor data.

### Unnormalized Table

```
CREATE TABLE StudentRecords (
    StudentID INT,
    StudentName VARCHAR(50),
    CourseID VARCHAR(10),
    CourseName VARCHAR(50),
    InstructorID INT,
    InstructorName VARCHAR(50),
    Skills VARCHAR(100) -- Multi-valued attribute, e.g., 'SQL,Python'
);
```

**Sample Data**:

|StudentID|StudentName|CourseID|CourseName|InstructorID|InstructorName|Skills|
|---|---|---|---|---|---|---|
|101|Alice|CS101|Database|201|Prof. Smith|SQL,Python|
|101|Alice|CS102|Algorithms|202|Prof. Jones|SQL,Python|
|102|Bob|CS101|Database|201|Prof. Smith|Java,C++|

**Issues**:

- `Skills` is multi-valued, violating 1NF.
- `StudentName` depends only on `StudentID` (partial dependency), violating 2NF.
- `CourseName → InstructorName`, a transitive dependency, violating 3NF.
- `InstructorID → CourseID`, where `CourseID` is not a superkey, potentially violating BCNF.

### Conversion Methods Between Normal Forms

Below are the steps to normalize the table to each normal form, with SQL implementations.

#### To 1NF

- **Objective**: Ensure atomic attributes and unique records.
- **Method**: Split `Skills` into a separate table; define a primary key (`StudentID`, `CourseID`).
- **SQL**:

```sql
CREATE TABLE Students (
    StudentID INT,
    StudentName VARCHAR(50),
    PRIMARY KEY (StudentID)
);

CREATE TABLE Courses (
    StudentID INT,
    CourseID VARCHAR(10),
    CourseName VARCHAR(50),
    InstructorID INT,
    InstructorName VARCHAR(50),
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID)
);

CREATE TABLE StudentSkills (
    StudentID INT,
    Skill VARCHAR(50),
    PRIMARY KEY (StudentID, Skill),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID)
);

-- Insert data
INSERT INTO Students VALUES (101, 'Alice'), (102, 'Bob');
INSERT INTO Courses VALUES (101, 'CS101', 'Database', 201, 'Prof. Smith'), 
                          (101, 'CS102', 'Algorithms', 202, 'Prof. Jones'),
                          (102, 'CS101', 'Database', 201, 'Prof. Smith');
INSERT INTO StudentSkills VALUES (101, 'SQL'), (101, 'Python'), (102, 'Java'), (102, 'C++');
```

#### To 2NF

- **Objective**: Remove partial dependencies (e.g., `StudentName` depends only on `StudentID`).
- **Method**: Already partially achieved in 1NF by separating `Students`. Ensure `CourseName`, `InstructorName` depend on the entire key (`StudentID`, `CourseID`).
- **SQL**: No further changes needed, as `Students` table handles `StudentName`, and `Courses` attributes depend on the composite key.

#### To 3NF

- **Objective**: Remove transitive dependencies (e.g., `CourseName → InstructorName`).
- **Method**: Create a separate `Instructors` table for `InstructorID → InstructorName` and `CourseInfo` for `CourseID → CourseName`.
- **SQL**:

```sql
CREATE TABLE Instructors (
    InstructorID INT,
    InstructorName VARCHAR(50),
    PRIMARY KEY (InstructorID)
);

CREATE TABLE CourseInfo (
    CourseID VARCHAR(10),
    CourseName VARCHAR(50),
    InstructorID INT,
    PRIMARY KEY (CourseID),
    FOREIGN KEY (InstructorID) REFERENCES Instructors(InstructorID)
);

CREATE TABLE Enrollments (
    StudentID INT,
    CourseID VARCHAR(10),
    PRIMARY KEY (StudentID, CourseID),
    FOREIGN KEY (StudentID) REFERENCES Students(StudentID),
    FOREIGN KEY (CourseID) REFERENCES CourseInfo(CourseID)
);

-- Insert data
INSERT INTO Instructors VALUES (201, 'Prof. Smith'), (202, 'Prof. Jones');
INSERT INTO CourseInfo VALUES ('CS101', 'Database', 201), ('CS102', 'Algorithms', 202);
INSERT INTO Enrollments VALUES (101, 'CS101'), (101, 'CS102'), (102, 'CS101');
```

#### To BCNF

- **Objective**: Ensure all determinants are superkeys (e.g., `InstructorID → CourseID` in `CourseInfo` violates BCNF if `InstructorID` is not a superkey).
- **Method**: Decompose `CourseInfo` to eliminate non-superkey determinants.
- **SQL**:

```sql
-- Assuming InstructorID → CourseID, CourseName (e.g., one instructor teaches one course)
CREATE TABLE InstructorCourses (
    InstructorID INT,
    CourseID VARCHAR(10),
    PRIMARY KEY (InstructorID),
    FOREIGN KEY (CourseID) REFERENCES CourseInfo(CourseID)
);

-- Update CourseInfo to remove InstructorID
ALTER TABLE CourseInfo DROP COLUMN InstructorID;
ALTER TABLE CourseInfo ADD PRIMARY KEY (CourseID);

-- Insert data (assuming one-to-one mapping for simplicity)
INSERT INTO InstructorCourses VALUES (201, 'CS101'), (202, 'CS102');
```

#### To 4NF

- **Objective**: Remove multi-valued dependencies (e.g., if `StudentID` independently determines `CourseID` and `Skill`).
- **Method**: `StudentSkills` and `Enrollments` already separate multi-valued dependencies (`StudentID → Skill` and `StudentID → CourseID`). No further changes needed unless new multi-valued dependencies arise.
- **SQL**: No changes required.

#### To 5NF

- **Objective**: Ensure no join dependencies (i.e., table cannot be decomposed without data loss).
- **Method**: Verify that `Enrollments` (junction table for `StudentID`, `CourseID`) is in 5NF by ensuring no non-trivial join dependencies exist. In this case, the table is already in 5NF, as it represents a many-to-many relationship without redundant projections.
- **SQL**: No changes required.

## Conversion Summary

- **Unnormalized to 1NF**: Split multi-valued attributes (`Skills`) into separate tables; define primary keys.
- **1NF to 2NF**: Separate partial dependencies (`StudentName` → `Students` table).
- **2NF to 3NF**: Remove transitive dependencies (`CourseName → InstructorName` → `Instructors`, `CourseInfo`).
- **3NF to BCNF**: Ensure all determinants are superkeys (`InstructorID → CourseID` → `InstructorCourses`).
- **BCNF to 4NF**: Eliminate multi-valued dependencies (already handled by `StudentSkills`, `Enrollments`).
- **4NF to 5NF**: Verify no join dependencies (confirmed for `Enrollments`).
- **Reverse Conversion (Denormalization)**: Combine tables or add redundant attributes (e.g., add `StudentName` to `Enrollments` for faster queries, as discussed in denormalization).

## Technical Notes

- **Relation to Prior Discussions**:
    - **Functional Dependency**: Drives normalization by identifying dependencies (e.g., `StudentID → StudentName`) for decomposition.
    - **ER Model**: The example aligns with the ER Model (e.g., `Students`, `Courses`, `Enrollments` as entities and relationships).
    - **DML**: Normalized tables support efficient DML operations but may require joins, impacting performance.
    - **Denormalization**: Reintroducing redundancy (e.g., `StudentName` in `Enrollments`) reverses normalization for performance.
    - **ACID Properties**: Normalization ensures consistency by preventing anomalies, supporting transaction integrity.
    - **Conflict Serializability**: Normalized schemas reduce conflicts in concurrent transactions by minimizing redundant updates.
- **Performance Considerations**: Higher normal forms reduce redundancy but increase query complexity (e.g., joins). Indexing (as discussed previously) can mitigate performance issues.
- **Tools**: MySQL and PostgreSQL support normalization through schema design tools like MySQL Workbench or pgAdmin. Refer to _MySQL Reference Manual_ ([https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)) or _PostgreSQL Documentation_ ([https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)) for implementation details.

## Revision Summary

- **Core Concept**: Normal Forms (1NF to 5NF) are rules to eliminate redundancy and anomalies in RDBMS schemas.
- **Types**: 1NF (atomic attributes), 2NF (no partial dependencies), 3NF (no transitive dependencies), BCNF (determinants are superkeys), 4NF (no multi-valued dependencies), 5NF (no join dependencies).
- **Conversions**: Iterative decomposition based on functional dependencies, with SQL used to create and populate tables.
- **Practical Example**: University database normalized from unnormalized to 5NF, addressing multi-valued, partial, and transitive dependencies.
- **Relevance to Prior Discussions**: Builds on functional dependencies, ER Model, DML, denormalization, and ACID properties.
- **Further Study**: Consult _Database System Concepts_ by Silberschatz, Korth, and Sudarshan for theoretical depth, or MySQL/PostgreSQL documentation for practical guidance.