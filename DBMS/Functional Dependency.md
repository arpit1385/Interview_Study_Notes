# Functional Dependency in DBMS

## Definition

A functional dependency (FD) in a Relational Database Management System (RDBMS) is a constraint between two sets of attributes in a relation (table), where one set (the determinant) uniquely determines the values of another set (the dependent). It is formally expressed as ( X \rightarrow Y ), where ( X ) and ( Y ) are sets of attributes, and for every value of ( X ), there is exactly one corresponding value of ( Y ).

## Characteristics

- **Purpose**: Defines relationships between attributes, forming the foundation for normalization (as discussed previously) by identifying dependencies that guide table decomposition.
- **Uniqueness Constraint**: Ensures that for any given value of the determinant ( X ), the value of ( Y ) is uniquely determined, supporting data integrity and consistency.
- **Types**:
    - **Trivial Functional Dependency**: When ( Y ) is a subset of ( X ), e.g., ( {A, B} \rightarrow A ). This is inherently true.
    - **Non-Trivial Functional Dependency**: When ( Y ) is not a subset of ( X ), e.g., ( EmployeeID \rightarrow Name ).
    - **Partial Dependency**: When a non-key attribute depends on part of a composite primary key, relevant in 2NF (referenced in prior normalization discussion).
    - **Transitive Dependency**: When a non-key attribute depends on another non-key attribute through a third attribute, relevant in 3NF.
- **Relation to Keys**: Functional dependencies are used to identify candidate keys (attributes that uniquely identify records) and primary keys, as discussed in prior notes on keys and relationships.

## Technical Notes

- **Formal Representation**: In a relation ( R ), if ( X \rightarrow Y ), then for any two tuples with the same value for ( X ), the values for ( Y ) must be identical. For example, in an `Employees` table, if ( EmployeeID \rightarrow Name ), every `EmployeeID` maps to exactly one `Name`.
- **Role in Normalization**: Functional dependencies guide the decomposition of tables to eliminate redundancy and anomalies:
    - Partial dependencies are removed in 2NF.
    - Transitive dependencies are addressed in 3NF.
    - Non-key determinants are resolved in BCNF (referenced in prior normalization notes).
- **Inference Rules (Armstrong’s Axioms)**:
    - **Reflexivity**: If ( Y \subseteq X ), then ( X \rightarrow Y ) (trivial FD).
    - **Augmentation**: If ( X \rightarrow Y ), then ( XZ \rightarrow YZ ) for any set ( Z ).
    - **Transitivity**: If ( X \rightarrow Y ) and ( Y \rightarrow Z ), then ( X \rightarrow Z ).
    - Additional derived rules include union, decomposition, and pseudo-transitivity.
- **Closure of Attributes**: The closure of a set of attributes ( X ), denoted ( X^+ ), is the set of all attributes functionally determined by ( X ). Used to identify candidate keys and verify dependencies.
- **Relation to Prior Discussions**:
    - **Normalization**: Functional dependencies are central to decomposing tables into normal forms (1NF, 2NF, 3NF, BCNF) to eliminate anomalies during DML operations (insert, update, delete).
    - **Keys and Relationships**: FDs define primary and candidate keys, ensuring referential integrity in relationships.
    - **Denormalization**: Reintroducing redundancy in denormalization (previously discussed) may violate strict FD constraints, requiring alternative mechanisms like triggers.
    - **ACID Properties**: FDs support consistency by ensuring predictable attribute relationships, aligning with transaction integrity.
- **Practical Implementation**: Modern RDBMS (e.g., MySQL, PostgreSQL) enforce functional dependencies implicitly through primary keys, foreign keys, and constraints, verified during schema design and DML operations.

## Example

Consider a table `StudentRecords` with attributes: `StudentID`, `CourseID`, `StudentName`, `CourseName`, `Instructor`.

- **Functional Dependencies**:
    - ( StudentID \rightarrow StudentName ): Each `StudentID` uniquely determines `StudentName`.
    - ( CourseID \rightarrow CourseName, Instructor ): Each `CourseID` determines `CourseName` and `Instructor`.
    - ( {StudentID, CourseID} \rightarrow StudentName, CourseName, Instructor ): The composite key determines all attributes.
- **Partial Dependency**: If `StudentName` depends only on `StudentID` (part of the composite key), it violates 2NF and requires decomposition.
- **Transitive Dependency**: If `CourseName \rightarrow Instructor`, and `CourseID \rightarrow CourseName`, then `CourseID \rightarrow Instructor` transitively, requiring resolution in 3NF.
- **Normalization Application**:
    - Split into `Students(StudentID, StudentName)` and `Courses(CourseID, CourseName, Instructor)` to eliminate partial and transitive dependencies.
    - Create a junction table `Enrollments(StudentID, CourseID)` to maintain relationships.

## Revision Summary

- **Core Concept**: Functional dependency (( X \rightarrow Y )) ensures one set of attributes uniquely determines another, foundational to database design.
- **Types**: Trivial, non-trivial, partial, and transitive dependencies, each relevant to specific normal forms.
- **Role**: Drives normalization to eliminate redundancy and anomalies, supports key identification, and ensures data integrity.
- **Relevance to Prior Discussions**: Underpins normalization (2NF, 3NF, BCNF), relates to keys, relationships, DML operations, and ACID properties.
- **Practical Use**: Guides schema design in RDBMS like MySQL or PostgreSQL to ensure efficient and consistent data management.
- **Further Study**: For detailed theory, refer to _Database System Concepts_ by Silberschatz, Korth, and Sudarshan. For implementation, consult _MySQL Reference Manual_ ([https://dev.mysql.com/doc/](https://dev.mysql.com/doc/)) or _PostgreSQL Documentation_ ([https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)).