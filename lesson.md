# **📘 Lesson Plan: SQL Data Definition (DDL) with DuckDB**
**In data science, data is the new oil. DDL is how we build the refinery.**

* **Module:** 1.3 Data Definition & Management 
* **Target Audience:** Adult Learners (Basic Python knowledge) 
* **Methodology:** Flipped Classroom / Code-Along / "I do, We do, You do"
* **Tools:** [DbGate](https://dbgate.org/), [DuckDB](https://duckdb.org/)

## **🎯 Learning Objectives (Bloom's Taxonomy)**

By the end of this session, learners will be able to:

1. **Identify** the hierarchical structure of a relational database (Database \> Schema \> Table) to organize data effectively.  
2. **Apply** `CREATE` statements with strict constraints (Primary/Foreign Keys) to **construct** a relational schema that enforces data integrity.  
3. **Execute** data pipeline commands (`COPY`) to **import** raw CSV data into structured tables for analysis.

## **⏱️ Agenda & Topic Prioritization**

| Time | Section | Focus | Top 3 Core Topics (Must Cover) | Optional Topics (Mention briefly or Skip) |
| ----- | ----- | ----- | ----- | ----- |
| **0:00 \- 0:55** | **1\. The Foundation** | **Identify** | **`CREATE TABLE` & Types:** Defining the physical container for data and choosing the right `VARCHAR` vs `INT`. | DuckDB internal storage (`.wal` files), Sequence objects. |
| **0:55 \- 1:05** | *Break* |  |  |  |
| **1:05 \- 2:00** | **2\. The Blueprint** | **Construct** | **Constraints (PK/FK):** The "Rules" of the database. How `PRIMARY KEY` and `FOREIGN KEY` prevent bad data. | Complex `CHECK` constraints (Regex), Composite Keys. |
| **2:00 \- 2:10** | *Break* |  |  |  |
| **2:10 \- 3:00** | **3\. The Pipeline** | **Execute** | **Data Import (`COPY`):** Moving external data (CSV) into the database to make it useful. | `DROP` vs `TRUNCATE`, Exporting to JSON. |





---
## **Section 1: The Foundation (Structure)**

**Goal:** Move from "What is a database?" to "I have a place to store data."

#### **1.1 The Narrative**

"In Python, you create variables like `my_data = []`. But when you turn off the computer, that data vanishes. To make data specific and permanent, we need a **Database**. Think of it like a physical File Cabinet:

* **The Database:** The Cabinet itself.  
* **The Schema:** A specific drawer (labeled 'HR' or 'Sales').  
* **The Table:** A folder inside that drawer.  
* **The Column:** The specific form fields in that folder (Name, Date, ID). Today, we aren't just coding; we are building the cabinet."

#### **1.2 Concept: Types Matter**

* **INTEGER:** Math-able numbers. (e.g., Age, Count).  
* **VARCHAR:** Text. (e.g., Name, Email).  
* **DATE:** Chronological points. (Don't store dates as text\!)

---

We will be using two modern, lightweight tools:

1. **DuckDB:** A fast, "in-process" database engine. Unlike traditional databases (like MySQL), DuckDB doesn't require a complex server setup. It stores everything in a single file on your computer, making it perfect for data science.  
2. **DbGate:** A user-friendly database manager. It provides a visual interface to see your tables, run queries, and manage your data.



---
#### **🟢 Activity 1: The "Profile" Table (Code-Along) (20 Mins)**
   
1. **Connecting to the Database:**
  
  * You can connect to any database using an SQL client or IDE like DbGate, which supports a broad range of popular SQL and NoSQL databases, including MySQL, PostgreSQL, SQL Server, MongoDB, SQLite, Oracle, Redis, CockroachDB, MariaDB, Amazon Redshift, ClickHouse, Cassandra, DuckDB, Firebird, and Firestore, with some premium features for cloud databases like Azure. It acts as a universal manager, allowing you to connect to and work with multiple database types from a single application.

  * For this lesson, we'll use DuckDB, a lightweight, in-process SQL database engine.

  * Open **DbGate**.  
  
  * Create a new connection to the DuckDB file provided in this repo.
     
```
   db/unit-1-3.db
```

> If you expand on the `unit-1-3.db`, you should see a few predefined schemas.  
> * The default schema is `main`.  
> * Any tables you create without specifying a schema will be in `main`.  
> * You can also create additional schemas to organize your tables.



2. **Creating a Schema:**

   We start by creating a new schema `lesson` to organise our tables.

```sql
CREATE SCHEMA lesson;
```

  * alternatively, we can also use the following to create a schema if it does not exist yet

```sql
CREATE SCHEMA IF NOT EXISTS lesson;
```


3. **Creating your first Table:**

  * We will be creating a table `users` in the `lesson` schema. The table will have the following columns:

- `id` - integer
- `name` - varchar
- `email` - varchar
  
```sql
CREATE TABLE lesson.users (  
  id INTEGER,  
  name VARCHAR,  
  email VARCHAR  
);
```

> If you just want to create tables in the default (`main`) schema, you can omit the `lesson.` prefix.


4. **Basic Data Entry (DML basics for testing DDL):**

  * We can insert data into the table using the `INSERT INTO` statement.
   
```sql
INSERT INTO lesson.users (id, name, email)  
VALUES (1, 'John Doe', 'john.doe@gmail.com');
```

  * This will insert a new row / record into the `users` table. You can insert multiple rows at once.

```sql
INSERT INTO lesson.users (id, name, email)
VALUES (2, 'Jane Doe', 'jane.doe@gmail.com'),
       (3, 'John Smith', 'john.smith@gmail.com');
```

> Insert two more rows with contiguously increasing `id` values, random `name`s and `email`s.

5. **Alter Tables**

  * We can alter the tables to add, rename or remove columns.

  * Add column 'start_date' to table users.

```sql
ALTER TABLE lesson.users ADD COLUMN start_date DATE;
```

Rename column 'id' to 'uid' in table classes.

```sql
ALTER TABLE lesson.users RENAME id TO uid;
```

6. **TRUNCATE (Emptying the table) vs. DROP (Deleting the table):**



```sql
ALTER TABLE lesson.users
DROP COLUMN email;
```

```sql
TRUNCATE TABLE lesson.users;
```

```sql
DROP TABLE lesson.users;
```
> **Tip: "Always run a SELECT before a DROP. Double-check your table name. It’s better to be slow and correct than fast and sorry."**

* **Q\&A:** Addressing connection issues and terminology (Database vs. Schema).  
* **Reflection:** Why is organizing data into schemas important in a production environment?

---
## **Section 2: Building Relationships & Constraints**

**Learning Objective:** By the end of this section, learners will be able to translate an Entity Relationship Diagram (ERD) into SQL tables using primary keys, foreign keys, and data constraints.

* **Theory Summary/Recap:**  
  * Constraints: Primary Keys (Uniqueness), Foreign Keys (Relationships), NOT NULL, CHECK, and DEFAULT.  
  * Understanding the School System ERD (Students, Teachers, Classes).

<details>
  <summary>Primary Keys, Foreign Keys, NOT NULL, CHECK, and DEFAULT</summary>
  
  * `primary key` or `unique` define a column, or set of columns, that are a unique identifier for a row in the table.
  * `primary key` constraints and unique constraints are identical except that a table can only have one primary key constraint defined, but many unique constraints and a primary key constraint also enforces the keys to not be NULL.
  * `foreign key` defines a column, or set of columns, that refer to a primary key or unique constraint from another table. The constraint enforces that the key exists in the other table.
  * `not null` specifies that the column cannot contain any NULL values. By default, all columns in tables are nullable.
  * `default` specifies a default value for the column when no value is specified.
  * `check` constraint allows you to specify an arbitrary boolean expression. Any columns that do not satisfy this expression violate the constraint.
</details>

* **Demo & Hands-on Workshop:**
 
1. **Creating Tables with Constraints:**  
```sql
CREATE TABLE lesson.teachers (
  id INTEGER PRIMARY KEY, -- primary key
  name VARCHAR NOT NULL, -- not null
  age INTEGER CHECK(age > 18 AND age < 70), -- check
  address VARCHAR,
  phone VARCHAR,
  email VARCHAR CHECK(CONTAINS(email, '@')) -- check
);

CREATE TABLE lesson.classes (
  id INTEGER PRIMARY KEY, -- primary key
  name VARCHAR NOT NULL, -- not null
  teacher_id INTEGER REFERENCES lesson.teachers(id) -- foreign key
);
```

2. **Exercise:** Complete the CREATE TABLE statement for the students table based on the ERD (Entity-Relationship Diagram) below.

> In a school system whose classes have students and teachers. Each student belongs to a single class. Each teacher may teach more than one class, but each class only has one teacher.

```dbml
Table students {
  id int [pk]
  name varchar
  address varchar
  phone varchar
  email varchar
  class_id int
}

Table teachers {
  id int [pk]
  name varchar
  age int
  address varchar
  phone varchar
  email varchar
}

Table classes {
  id int [pk]
  name varchar
  teacher_id int
}

Ref: students.class_id > classes.id // many-to-one

Ref: classes.teacher_id > teachers.id // many-to-one
```

* **Q\&A:** Troubleshooting foreign key errors and understanding "CHECK" logic.  
* **Reflection:** How do constraints prevent "bad data" from entering your analysis pipeline?


---
## **Section 3: Data Management & Performance**

**Learning Objective:** By the end of this section, learners will be able to import, update and export data, improve query performance with indexes, explain differences between tables and views. 

* **Theory Summary/Recap:**
  * The COPY command for CSV/JSON handling.
  * What are Indexes? (Think of a book index for speed).  
  * Tables vs. Views (Physical storage vs. Virtual queries).  
  

<details>
  <summary>What are Indexes?</summary>
  
  * Indexes are used to improve the performance of queries. They are not required but are recommended for tables with many rows. They are used to _retrieve data from the database more quickly than otherwise_. Indexes are created using one or more columns of a database table. The users cannot see the indexes, they are just used to speed up searches/queries.
</details>

<details>
  <summary>Tables vs. Views</summary>
  
  * Tables and views are both ways to store data. What is the difference between them? A table is a physical copy of the data (materialized), while a view is a virtual copy of the data. A view is a query that is run on the fly when you access the view. A view is not stored in the database, but the query that defines the view is stored in the database.
</details>



* **Demo & Hands-on Workshop:**

1. **Importing Data:**

  * We can import data from a CSV file into a table.

```sql
COPY lesson.teachers FROM '<full directory path>' (AUTO_DETECT TRUE);
```

> **Note:** For data import, use the full directory path to the CSV files, e.g. `/Users/fengfeng/Dev/6m-data-1.3-sql-basic-ddl/data/teachers.csv`

2. **Exercise:**

  * Import data for `classes` and `students` tables. 


3. **Updating Data:**

  * We can update the data in the table using the `UPDATE` statement.

  * Let's say `Linda Garcia` changed her email to `linda.g@example.com`, we can update the `email` of the student with id 4 (her id) to `linda.g@example.com`. The `WHERE` clause is used to specify which rows to update.

```sql
UPDATE lesson.students
SET email = 'linda.g@example.com'
WHERE id = 4;
```
  > This is DML, we will learn more in next lesson. 

4. **Exporting Data:**

  * Let's export the data from the student table into a CSV file delimited with `|`.       Remember to prepend the full directory path to the CSV file.

```sql
COPY (SELECT * FROM lesson.students) TO '<full directory path>' WITH (HEADER 1, DELIMITER '|');

-- example <full directory path>: /Users/fengfeng/Dev/6m-data-1.3-sql-basic-ddl/data/students_new.csv
```

  * We can also export the data into a JSON file (you will learn more about JSON in Module 2).

```sql
COPY (SELECT * FROM lesson.students) TO '<full directory path>';

-- example <full directory path>:  /Users/fengfeng/Dev/6m-data-1.3-sql-basic-ddl/data/students.json
```

5. **Exercise:**

  * Repeat the above steps for the `teachers` & `classes` tables.



6. **Creating Indexes:**

   <details>
   <summary>Indexes</summary>
  
   * Indexes are used to improve the performance of queries. They are not required but are recommended for tables with many rows. They are used to _retrieve data from the database more quickly than otherwise_. Indexes are created using one or more columns of a database table. The users cannot see the indexes, they are just used to speed up searches/queries.
   </details>


  

```sql
-- Create a unique index 'teachers_name_idx' on the column name of table teachers.

CREATE UNIQUE INDEX teachers_name_idx ON lesson.teachers(name);
```


```sql
-- Create index 'students_name_idx' that allows for duplicate values on the column name of table students.

CREATE INDEX students_name_idx ON lesson.students(name);
```

```sql
-- View indexes for a specific table
SELECT index_name, sql 
FROM duckdb_indexes 
WHERE table_name = 'students';
```

7. **Tables vs Views:**

   <details>
   <summary>Tables and Views</summary>
  
   * Tables and views are both ways to store data. What is the difference between them? A table is a physical copy of the data (materialized), while a view is a virtual copy of the data. A view is a query that is run on the fly when you access the view. A view is not stored in the database, but the query that defines the view is stored in the database.
   </details>

  * We will be creating a view `students_view` in the `lesson` schema. The view will have the following columns:

  - `id` - integer
  - `name` - varchar
  - `email` - varchar

```sql
CREATE VIEW lesson.students_view AS
SELECT id, name, email
FROM lesson.students;
```

8. **Exercise:**

  * Create a view `teachers_view` with the same columns as `students_view` but for the `teachers` table.
   


* **Q\&A:** When should you *not* use an index? Difference between dropping a table vs. deleting data.  
* **Reflection:** How does using a View simplify the work for a Data Analyst who only needs specific columns?




---

## **Self-Study & Advanced Parts (Optional)**
### **Local Environment Setup**

If you want to create the database file from scratch:

1. Create a new conda environment from `environment.yml`
```
  conda env create -f environment.yml
```
2. Activate the conda environment
```
  conda activate ddb
```   
3. Run [create_duckdb.py](./db/create_duckdb.py) to create the database file.
```
  python db/create_duckdb.py
```

