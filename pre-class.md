# 📚 Pre-Class: SQL Data Definition Language (DDL)

**Estimated Time:** 30–40 minutes
**Prerequisites:** Lesson 1.2 — Data Modeling & Schema Design

> In Lesson 1.2 you designed database schemas using DBML and dbdiagram.io. This lesson is where you *build* those schemas for real — writing the actual SQL commands that create tables, define columns, and enforce constraints in a live database.

Complete the steps below **before your class session**.

---

## Step 1 — Watch the Video (10 min)

🎬 [SQL DDL — The Blueprint for Data](https://youtu.be/adACCuMFjac)

An introduction to what DDL is, why it exists as a separate category of SQL, and a walkthrough of the core commands you'll use in class.

---

## Step 2 — Read the Core Concepts (15 min)

### What is a Database — and How is it Structured?

Think of a database as a highly organised digital filing cabinet:

- **Database** — the cabinet itself (e.g., `School_System`)
- **Schema** — a drawer inside the cabinet, used to group related tables (e.g., `Enrollment_Data`)
- **Table** — a specific spreadsheet inside the drawer (e.g., `Students`)
- **Columns (Fields)** — the categories of data (e.g., Name, Age, Email)
- **Rows (Records)** — individual entries (e.g., "John Doe, 25, john@example.com")

### What is DDL?

**Data Definition Language (DDL)** is the subset of SQL used to **build and modify the structure** of a database — not the data itself. Think of it as the architect's work: you design and build the rooms before moving furniture in.

| DDL Command | What it does |
|---|---|
| `CREATE SCHEMA` | Creates a new folder/namespace in the database |
| `CREATE TABLE` | Defines a new table and its columns |
| `ALTER TABLE` | Adds or changes columns in an existing table |
| `DROP TABLE` | Deletes a table entirely ⚠️ **cannot be undone** |

**Data Types** — when creating columns, you must tell the database what kind of data goes in each:

| Type | Use for |
|---|---|
| `INTEGER` | Whole numbers (e.g., age, quantity) |
| `VARCHAR(n)` | Short text up to n characters (e.g., name, email) |
| `TEXT` | Long free-form text |
| `DATE` | Calendar date (e.g., 2026-01-01) |
| `BOOLEAN` | True or false |

---

## Step 3 — Tool Setup (10 min)

In class we use two tools. Install both before the session.

### DuckDB
[DuckDB](https://duckdb.org/) is the database engine we'll use — it runs entirely on your laptop with no server required, making it ideal for data science work.

<details>
<summary>📖 Why DuckDB? (expand for details)</summary>

Most databases need a separate server running in the background. DuckDB is **in-process** — it runs inside your Python script or DbGate session directly. This means:
- No installation of a server, no configuration
- The entire database lives in a single `.db` file you can copy or share
- Optimised for the kind of analytical queries (aggregations, joins across millions of rows) that data scientists run most often

</details>

### DbGate — Community Edition (free)
[Download DbGate](https://www.dbgate.io/download-community/) — a visual database manager that lets you browse tables, run SQL queries, and inspect results without writing everything in a terminal.

<details>
<summary>📖 What can DbGate do? (expand for details)</summary>

- Connect to your DuckDB `.db` file with a few clicks
- Browse tables and their data like a spreadsheet
- Write and run SQL with auto-completion
- Export query results to CSV or Excel

</details>

**Setup steps:**
1. Download and install DbGate (Community edition, free)
2. Download the `unit-1-3.db` file from the [course repository](https://github.com/su-ntu-ctp/6m-data-1.3-sql-basic-ddl/blob/main/db/)
3. Open DbGate → New Connection → DuckDB → point to your `.db` file

---

## Step 4 — Check Your Understanding

Try to answer these before class. Write your answers down, then check below.

1. If I want to add a "Phone Number" column to an existing table, which DDL command do I use?
2. What is the difference between a Schema and a Table?
3. Why might a Data Scientist prefer DuckDB over a traditional database server like PostgreSQL?

<details>
<summary>Suggested answers</summary>

**Q1:** `ALTER TABLE` — it modifies the structure of an existing table without destroying it or its data.

**Q2:** A Schema is a namespace or folder that groups related tables together (e.g., all tables related to `enrollment` live in the `enrollment` schema). A Table is the actual data structure within that schema — rows and columns. One Schema can contain many Tables.

**Q3:** DuckDB requires no server setup or administration — it runs as a single file on your laptop. For a data scientist doing local analysis, this means less overhead, easier reproducibility (just share the `.db` file), and better performance on the analytical queries (aggregations, large scans) that are common in data science work.

</details>

---

If you get stuck during setup, post a screenshot in the **#questions** Discord channel before class.
