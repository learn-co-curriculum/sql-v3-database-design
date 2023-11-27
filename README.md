# Database Design

## Learning Goals

- Define data anomalies that result from data redundancy.
- Define database normal forms:
  - First Normal Form (1NF)
  - Second Normal Form (2NF)
  - Third Normal Form (3NF)
- Apply the normalization process to evolve an un-normalized table into 3NF.

## Introduction

The goal of relational-database design is to develop
a set of tables that (1) accurately reflect the real world domain
and (2) store all necessary information without unnecessary redundancy.

A database anomaly is a flaw in the database design,
usually caused by data redundancy, that results in incorrect data.
Anomalies also result if the design does not
match with the real-world use cases the database claims to model.

In this lesson, we will explore common data anomalies that occur due to poor design.
We will apply the normalization process to evolve a poorly designed table
into a set of well-designed tables that avoid such data anomalies.

## Data Anomalies

It is important to understand how a poor database design may result in data anomalies.

Assume we need a database to store information about people who rent
monthly parking spaces at various parking garages.
The initial design uses a single table `parking_permit` to store the information.

| email      | style      | fee  | garage  |
|------------|------------|------|---------|
| abc@co.com | sedan      | 50   | A       |
| xyz@co.com | sedan      | 50   | A, B    |
| pqr@co.com | truck      | 75   | C       |
| lmn@co.com | truck      | 75   | A, B, C |
| def@co.com | motorcycle | 25   | C       |

- The primary column `email` identifies a person acquiring a parking permit.
- The table stores the person's vehicle style, as well as the parking fee and garage permits.
- The monthly parking fee is based on the vehicle style (sedan, truck, etc), as larger styles take up more space.
- A person may purchase permits to park in multiple parking garages (garage A, B, or C).

The table above contains redundant data, i.e. the vehicle style and fee is duplicated among multiple rows.
Another issue is the `garage` column, which is a multivalued attribute.
The current table design will lead to **data anomalies** when we try to insert, update, or delete data:

- **Insertion Anomaly:**  We need to add a new vehicle style and fee (minivan, $65),
  but we don't yet have a customer who owns a vehicle with that style.
  The current design prevents this since `email` is the primary key.
- **Update Anomaly:** We need to change the fee for sedans to $55. We
  accidentally update the fee in the first row but not the second row, resulting in inconsistent data regarding sedan fees.
- **Deletion Anomaly:** We need to delete the row with email `def@co.com`, but
  deleting the row also causes all motorcycle fee data to be lost.


It is also awkward to write queries and certain update/delete operations for the table, for example:

- List the email of people parking in garage B.
- Count the number of sedans parked in garage B.
- Count the number of garages that have a sedan parked in them.
- The person with email `lmn@co.com` no longer wants to park in garage `B`.
- The person with email `xyz@co.com` wants to also park in garage `C`.

## Database Normalization

Database normalization is a process that takes a poorly designed database
and evolves it into a good design, i,e, a design that avoids data anomalies and supports
the required functionality.   The normalization process was proposed
by British computer scientist Edgar F. Codd in 1970.

**Normalization Goals**:
- Minimize data redundancy to avoid anomalies and reduce memory space.
- Simplify the enforcement of referential integrity constraints (foreign keys).
- Make it easier to maintain data (i.e. insert, update, and delete operations).

How do we achieve normalization?  We ensure the database schema
(tables and relationships) conforms to a set of **normal forms**.
There are numerous normal forms; however, we will focus on achieving
**Third Normal Form**:

- **First normal form (1NF)**: A table is a relation and has no multivalued attributes.
- **Second normal form (2NF)**: A table is in 1NF and has no partial dependencies.
- **Third normal form (3NF)**: A table is in 2NF and has no transitive dependencies.


## First Normal Form (1NF)

Function
: A **function** is a specific type of relation in which each input value
has one and only one output value. An input is the independent value,
and the output value is the dependent value, as it depends on the value of the input.

A database table represents a set of functions that can't be computed
using an algorithm. Instead, each function is stored as a column in the table.
So instead of calculating a value using an algorithm, 
we get the value from a table column based on the primary key.

For example, there is no algorithm to calculate date of birth from a social security number.
But each social security number has exactly one date of birth associated with it.
We can design a table that has a column `ssn` as the primary key, and a column
`date_of_birth` that contains a single date value.  The `date_of_birth` column acts as a lookup
function using the `ssn` primary key value as input.

**First Normal Form (1NF)**
: A table is in 1NF if the table is a relation (i.e. each row has a primary key,
each column contains data of the same type, all values are atomic) and there are no multivalued attributes.

First normal form (1NF) ensures that each table column represents a function that
maps a primary key value to a single output value.

Consider whether the `parking_permit` table is in 1NF:

| email      | style      | fee  | garage  |
|------------|------------|------|---------|
| abc@co.com | sedan      | 50   | A       |
| xyz@co.com | sedan      | 50   | A, B    |
| pqr@co.com | truck      | 75   | C       |
| lmn@co.com | truck      | 75   | A, B, C |
| def@co.com | motorcycle | 25   | C       |

- Each row is unique and has a primary key `email`.
- Each column has a unique name and contains data of the same type.
- However, the `garage` attribute is multivalued, thus the table is **not** in 1NF.

The `garage` column does not represent a function on the primary key column `email`, since
an email may be related to multiple garages.

The alternative table version shown below is also **not** in 1NF.
While the individual table cells appear to hold atomic values,
the `garage` attribute is still a multivalued attribute that is
simply stored across multiple columns rather than stored as a list in a single column.
What if we need to add a fourth or fifth garage?
Notice also many rows have null values in the columns, which is often an indication of a poor design.

| email      | style      | fee  | garage_1 | garage_2  | garage_3  |
|------------|------------|------|----------|-----------|-----------|
| abc@co.com | sedan      | 50   | A        | null      | null      |
| xyz@co.com | sedan      | 50   | A        | B         | null      |
| pqr@co.com | truck      | 75   | C        | null      | null      |
| lmn@co.com | truck      | 75   | A        | B         | C         | 
| def@co.com | motorcycle | 25   | C        | null      | null      |

### Evolving A Table to 1NF (composite primary key)

We need to evolve the `parking_permit` table design to
support the multivalued dependency between `email` and `garage`.
Since email `xyz@co.com` is associated with two parking garages, the new table
will store two rows, one for garage `A` and one for garage `B`, as shown below.
The person with email `lmn@co.com` will require 3 rows, and so on.

| email      | style      | fee | garage |
|------------|------------|-----|--------|
| abc@co.com | sedan      | 50  | A      |
| xyz@co.com | sedan      | 50  | A      |
| xyz@co.com | sedan      | 50  | B      |
| pqr@co.com | truck      | 75  | C      |
| lmn@co.com | truck      | 75  | A      |
| lmn@co.com | truck      | 75  | B      |
| lmn@co.com | truck      | 75  | C      |
| def@co.com | motorcycle | 25  | C      |


Our redesigned  `parking_permit` table no longer has multivalued attributes.
However, we can't use  `email` as the primary
key since we have two rows with email `xyz@co.com` and three rows with email `lmn@co.com`.
We need to define a **composite primary key** `(email, garage)` to model the multivalued attribute. 

The redesigned table `parking_permit` is in 1NF:

- Each row is unique and has a primary key `(email, garage)`.
- Each column has a unique name and contains atomic data of the same type.
- There are no multivalued attributes, thus each non-key column represents a function on the primary key `(email, garage)`.

However, notice the new table results in data redundancy.
The values in columns `style` and `fee`
are duplicated across multiple rows for email `xyz@co.com` and `lmn@co.com`.
As we saw earlier in the lesson, data redundancy often leads to data anomalies.

## Second Normal Form (2NF)

**Second Normal Form (2NF)**
: A table is in 2NF if it is in 1NF and there are no partial dependencies.

- A table with a single-column primary key has no partial dependencies.
- A table with a composite primary key (i.e. multiple columns) has a partial dependency
  if a non-key attribute is determined by part of the primary key.

For example, our redesigned table has composite primary key `(email, garage)`
and non-key attributes `style`, and `fee`.
The non-key attributes are determined by (i.e. related to) the person renting the parking permit
(uniquely identified by `email`). Since some rows for garage `A`
have different values for `style` and `fee`, the attributes are not related to `garage`.

| email      | style      | fee | garage |
|------------|------------|-----|--------|
| abc@co.com | sedan      | 50  | A      |
| xyz@co.com | sedan      | 50  | A      |
| xyz@co.com | sedan      | 50  | B      |
| pqr@co.com | truck      | 75  | C      |
| lmn@co.com | truck      | 75  | A      |
| lmn@co.com | truck      | 75  | B      |
| lmn@co.com | truck      | 75  | C      |
| def@co.com | motorcycle | 25  | C      |

The table is not in 2NF since and there are partial dependencies: non-key
attributes  `style`, and `fee` are determined by  `email` but not `garage`.

### Evolving A Table to 2NF (remove partial dependencies)

We need to evolve the design to remove the partial dependencies.  Since `style`, and `fee`
depend only on `email`, we will create a separate table `vehicle_owner` to hold that dependency.
The new design results in two tables:

- Table `vehicle_owner` has primary key `email` and non-key attributes `style`, and `fee`.
- Table `parking_permit` has composite primary key `(email, garage)`.  
  - The `email` column is a foreign key into the `vehicle_owner` table.



#### vehicle_owner table 

| email      | style      | fee |
|------------|------------|-----|
| abc@co.com | sedan      | 50  |
| xyz@co.com | sedan      | 50  |
| pqr@co.com | truck      | 75  |
| lmn@co.com | truck      | 75  |
| def@co.com | motorcycle | 25  |


#### parking_permit table 

| email      | garage  |
|------------|---------|
| abc@co.com | A       |
| xyz@co.com | A       |
| xyz@co.com | B       |
| pqr@co.com | C       |
| lmn@co.com | A       |
| lmn@co.com | B       |
| lmn@co.com | C       |
| def@co.com | C       |


- Table `vehicle_owner` is in 2NF since it is in 1NF and there are no partial dependencies (single column primary key)
- Table `parking_permit` is in 2NF since it is in 1NF and there are no partial dependencies (no non-key attributes)

## Third Normal Form (3NF)

**Third Normal Form (3NF)**
: A table is in 3NF if it is in 2NF and there are no transitive dependencies.

A transitive dependency occurs when a non-key attribute is determined by another non-key attribute.

Consider the redesigned `vehicle_owner` table. If we change the vehicle style in the first row to `motorcycle`,
the `fee` would need to change as well.  

| email      | style      | fee |
|------------|------------|-----|
| abc@co.com | sedan      | 50  |
| xyz@co.com | sedan      | 50  |
| pqr@co.com | truck      | 75  |
| lmn@co.com | truck      | 75  |
| def@co.com | motorcycle | 25  |


The table is in 2NF since there are no partial dependencies
(the primary key is the single column `email`).  However,
the table is not in 3NF since the non-key attribute `fee` is determined by
the non-key attribute `style`, rather than the primary key `email`. 


### Evolving A Table to 3NF (remove transitive dependencies)

We need to evolve the design of the `vehicle_owner` table to remove the transitive dependency `email->style->fee`.
Since `fee` depends only on `style`, we will create a separate table `vehicle_fee` to hold that dependency.
The new design results in an additional table `style_fee`:

- Table `style_fee` has primary key `style` and non-key attribute `fee`.
- Table `vehicle_owner` has primary key `email` and non-key attribute `style`. 
  - The `style` column is a foreign key into the `style_fee` table.
- Table `parking_permit` has composite primary key `(email, garage)`.
  - The `email` column is a foreign key into the `vehicle_owner` table.

#### style_fee table

| style      | fee |
|------------|-----|
| sedan      | 50  |
| truck      | 75  |
| motorcycle | 25  |


#### vehicle_owner table

| email      | style      |
|------------|------------|
| abc@co.com | sedan      |
| xyz@co.com | sedan      |
| pqr@co.com | truck      |
| lmn@co.com | truck      |
| def@co.com | motorcycle |


#### parking_permit table

| email      | garage  |
|------------|---------|
| abc@co.com | A       |
| xyz@co.com | A       |
| xyz@co.com | B       |
| pqr@co.com | C       |
| lmn@co.com | A       |
| lmn@co.com | B       |
| lmn@co.com | C       |
| def@co.com | C       |


- Table `style_fee` is in 3NF since it is in 2NF and there are no transitive dependencies (`fee` is determined by primary key `style`)  
- Table `vehicle_owner` is in 3NF since it is in 2NF and there are no transitive dependencies (`style` is determined by primary key `email`)  
- Table `parking_permit` is in 3NF since it is in 2NF and there are no transitive dependencies (there are no non-key attributes)  

The **Entity Relationship Diagram** below shows the final database design. Primary keys are displayed with bold font.

![parking erd](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-database-design/parking_erd.png)

Given the normalized database design, consider the things that caused anomalies with the original table:

- It is easy to add a new vehicle style and fee (minivan, $65).
- It is easy to change the sedan fee to $55.
- It is easy to delete the vehicle owner with email `def@co.com` without losing data about motorcycle fees.
- It is easy get the emails of people parking in garage B.
- It is easy to count the sedans parked in garage B.
- It is easy to count the garages with sedans.
- It is easy to delete email `lmn@co.com` from parking in garage `B`.
- It is easy to add email `xyz@co.com` to park in garage `C`.


## Conclusion

Database normalization is a process that evolves a poorly designed database
into a good design, i,e, one that avoids data anomalies and supports
the required functionality:

- **First normal form (1NF)**: Evolve a table with multivalued attributes into a table with a composite primary key.
- **Second normal form (2NF)**: Extract each partial dependency into a new table.  Add a foreign key to the new table.
- **Third normal form (3NF)**: Extract each transitive dependency into a new table. Add a foreign key to the new table.

## Resources

- [Database Normalization](https://en.wikipedia.org/wiki/Database_normalization)
