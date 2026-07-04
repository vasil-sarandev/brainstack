# Normalization

#deep-dive

Database normalization is the process of structuring a relational database in accordance with a series of so-called normal forms in order to reduce data redundancy and improve data integrity.

---
## 1NF (First Normal Form)

Ensures that the database table is organized such that each column contains atomic (indivisible) values, and each record is unique. This eliminates repeating groups, thereby structuring data into tables and columns.

Example Violation:
![1nf-violation](1nf-violation.png)

Courses attribute (column) is not an atomic value. It is further divisible.

---
## 2NF (Second Normal Form)

Builds on 1NF by ensuring that we remove redundant data from a table that is being applied to multiple rows by placing them in separate tables. It requires all non-key attributes to be fully functional on the primary key.

Example Violation:
![2nf-violation](2nf-violation.png)

Instructor only depends on part of the primary key (which is StudentID, CourseID). Therefore it is in violation of 2NF.

---
## 3NF (Third Normal Form)

Extends 2NF by ensuring that all non-key attributes are not only fully functional on the primary key but also independent of each other. This eliminates transitive dependency.

Example Violation:
![3nf-violation](3nf-violation.png)

Primary key is CourseID. InstructorName depends on InstructorID though so this table is a violation of 3NF.

---
## BCNF (Boyy-Codd Normal Form)

A refinement of 3NF that addresses anomalies not handled by 3NF. It requires every determinant to be a candidate key, ensuring even stricter adherence to normalization rules.

---
## 4NF (Fourth Normal Form)

Addresses multi-valued dependencies. It ensures that there are no multiple independent multi-valued facts about an entity in a record.

Example Violation:
![4nf-violation](4nf-violation.png)

There's a multi-valued dependency here. StudentName, Course, and Sport all depend on StudentID.

The fix to satisfy 4NF: break this down into three tables:

- Student (id, name)
- Student_course (id, course)
- Student_sport (id, sport)

---
## 5NF (Fifth Normal Form)

Also known as “Projection-Join Normal Form” (PJNF), It pertains to the reconstruction of information from smaller, differently arranged data pieces.

---
## 6NF (Sixth Normal Form)

Theoretical and not widely implemented. It deals with temporal data (handling changes over time) by further decomposing tables to eliminate all non-temporal redundancy.
