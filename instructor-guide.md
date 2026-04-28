# 🎓 Instructor Guide — Lesson 1.3: SQL Basic — DDL with DuckDB

> **Branch:** `feature/instructor-guide`
> **Audience:** Instructors and teaching assistants
> **Companion to:** `lesson.md`, `pre-class.md`, `assignment.md`

---

## 1. Lesson Overview & Instructor Objectives

| | |
|---|---|
| **Duration** | 3 hours |
| **Format** | Flipped Classroom + Code-Along ("I do, We do, You do") |
| **Tools** | DuckDB + DbGate |
| **Learner entry point** | Completed 1.2 (ERD / normalization); no prior SQL experience |

By the end of this lesson learners should be able to:

1. Navigate the Database → Schema → Table hierarchy in DbGate.
2. Write `CREATE TABLE` statements with appropriate data types and constraints.
3. Use `COPY` to import CSV data into a table.
4. Create indexes and views to optimise and simplify data access.

**Instructor's primary job:** Learners are writing real SQL for the first time. The "I do, We do, You do" structure is critical — resist the urge to give answers early. Let learners encounter constraint errors and read the error messages; that's where the learning happens.

---

## 2. Concept Analogies

### The File Cabinet (Database Hierarchy)

> "In Python, you store data in a variable — but when you close the script, it's gone. A database is the permanent home. Think of it as a filing cabinet:
> - The **database** is the entire cabinet.
> - A **schema** is a labelled drawer (HR, Sales, Finance).
> - A **table** is a folder inside that drawer.
> - A **column** is a specific field on the form inside that folder."

Follow-up question: "If you're building a payroll system and a customer management system in the same database, why would you put them in different schemas?" → Organisation, access control, avoiding name collisions.

---

### Data Types — "The Right Container"

> "You wouldn't store soup in a paper bag or carry water in a bucket with holes. Data types are containers — choose the wrong one and data leaks or gets distorted."

| Type | Container | Why It Matters |
|------|-----------|---------------|
| `INTEGER` | A counting tray — only whole numbers fit | Can do arithmetic; can't store "thirty-four" |
| `VARCHAR` | A flexible envelope | Works for names, emails, but *don't store dates as VARCHAR* |
| `DATE` | A calendar slot | Enables date math: `hire_date + 90 days = probation_end_date` |
| `BOOLEAN` | A light switch — on or off | Perfect for flags: `is_active`, `is_verified` |

**The "Don't store dates as text" moment:** Write `'2023-01-15'` on the board and ask: "Can I add 7 days to this text string?" (No.) "Can I sort these chronologically without them going alphabetical?" (You can, but only because the format happens to sort correctly — it's fragile.) "That's why we have DATE." This lands the concept better than any definition.

---

### Constraints — "The Database Bouncer"

> "In Excel, you can type 'banana' in a column called Price. Excel doesn't care. A database *cares*. Constraints are the bouncers at the door. They check every row before it gets in."

| Constraint | What the Bouncer Checks |
|------------|------------------------|
| `PRIMARY KEY` | "Have you been here before? IDs must be unique — no duplicates allowed." |
| `NOT NULL` | "You must show ID to enter — NULL is not a valid answer." |
| `FOREIGN KEY` | "Are you on the guest list? You can't reference a customer that doesn't exist." |
| `CHECK` | "You must be over 18 to enter — the value must pass this condition." |

**The "cheaper at the door" principle:** Rejecting bad data at entry is always cheaper than finding and fixing it downstream. A constraint violation at INSERT is a one-line error. The same bad data discovered in a production report two months later means tracing it back through the entire pipeline.

---

### COPY — "The Industrial Vacuum Cleaner"

> "Typing `INSERT INTO` one row at a time is fine for 5 rows. It's impossible for 1 million rows. The `COPY` command is the industrial vacuum cleaner of SQL — it inhales an entire CSV file in seconds."

This sets up the practical reality of data engineering: real datasets arrive as files, not manual inserts. The `COPY` command is the bridge between the file system and the database.

---

### Views — "The Custom Window"

> "A view is like a custom window cut into a wall. The data behind the wall doesn't change, but you can see exactly the slice you want. And you can give different people different windows — a data analyst sees only the columns they need, without touching the raw table."

Contrast with a table: "A table is the data itself, physically stored. A view is just a saved question — every time you query it, it runs the underlying query fresh."

---

## 3. Real-World Use Cases

### Schemas in Production

Large companies use schemas to separate concerns within a single database instance:
- `hr.employees` — only HR team has write access
- `sales.transactions` — read by BI tools, written by the sales system
- `analytics.monthly_summary` — populated by nightly batch jobs

This separation prevents accidental overwrites and enforces least-privilege access.

### Constraints Preventing Real Disasters

A common real-world scenario: a healthcare company had a `patients` table without a `NOT NULL` constraint on `date_of_birth`. A data import ran with blank dates — 40,000 patients with no age. The downstream model that calculated drug dosages by age group broke silently. The bug was in production for three weeks before anyone noticed. A simple `NOT NULL` constraint would have blocked the import immediately.

### COPY in Data Engineering

Data engineers at companies like Airbnb, Shopify, and Stripe ingest gigabytes of CSV/JSON logs daily using `COPY`-equivalent commands into data warehouses. The `COPY` command in this lesson is conceptually identical to the bulk load commands in Snowflake, BigQuery, and Redshift. Learning it in DuckDB now directly transfers to enterprise tools.

---

## 4. Activity Facilitation Notes

### Part 1: The Profile Table Code-Along (20 min)

**"I do" phase (5 min):** You type the CREATE TABLE, run it, show the table in DbGate. Narrate every line out loud: "I'm defining a column called `id` as an INTEGER — that means whole numbers only."

**"We do" phase (10 min):** Learners type the INSERT statements alongside you. Do one row together, then let them add two more independently.

**TRUNCATE vs DROP moment:** This is easy to forget but critically important. Write both on the board:
- `TRUNCATE` — empties the table, keeps the structure. Recoverable (in some databases, not DuckDB).
- `DROP` — destroys the table. **Unrecoverable.**

Then say: *"Always run a SELECT before a DROP. Check the table name three times. I have seen senior engineers drop the wrong table in production. It is not a fun day."*

---

### Part 2: The Missing Link Challenge (25 min)

**Part A — Code-Along with constraints:**

The `CHECK(CONTAINS(email, '@'))` constraint is a good conversation starter: "What happens if someone enters 'notanemail'?" (The database rejects the INSERT.) "What's the risk with this constraint?" (It accepts `@` which technically passes but isn't a valid email.) → Leads to the idea that CHECK constraints are a baseline, not a full validation system.

**Part B — Student table challenge:**

Most learners will write the FK reference incorrectly on the first attempt. The error message from DuckDB is helpful — read it together:
> "Referenced table `lesson.classes` not found"

This usually means they haven't created the `classes` table yet, OR they're referencing `classes` without the schema prefix. Both are valuable lessons about dependency order and naming.

**Dependency order is a key concept:** You cannot create `classes` before `teachers` (because `classes.teacher_id` references `teachers.id`). You cannot create `students` before `classes`. This is the real-world challenge of database migration scripts — order matters.

---

### Part 3: The CSV Dump (Code-Along)

**The most common failure point:** File path issues. Learners on Windows use backslashes; DuckDB expects forward slashes (or raw strings). Have this note ready:
- Windows: `C:/Users/yourname/Dev/...` (forward slashes) or `C:\\Users\\yourname\\Dev\\...`
- Mac/Linux: `/Users/yourname/Dev/...`

**After the import, always run:**
```sql
SELECT COUNT(*) FROM lesson.teachers;
SELECT * FROM lesson.teachers LIMIT 5;
```
This confirms the data arrived correctly. This "verify after load" habit is professional practice.

**The "why not INSERT?" question is worth spending 2 min on:**
Show the learners the CSV — if it has 500 rows, ask "how long would it take to type 500 INSERT statements?" Then show how `COPY` loads all 500 rows in one command. The difference in *approach* — bulk loading vs. row-by-row — is fundamental to understanding how real data systems work.

---

### Part 4: Index & View Code-Along

**Indexes:** Use the book analogy: "A book without an index makes you read every page to find your answer. A database without an index on a frequently-queried column does the same — it scans every row." Then ask: "When should you *not* use an index?" → On columns that are rarely queried (indexes slow down writes), and on very small tables where a full scan is faster than looking up the index.

**Views:** After creating `students_view`, query it: `SELECT * FROM lesson.students_view;`. Ask: "Does the view store the data or the query?" Then update a student's name directly in the `students` table and query the view again — the view reflects the change immediately. This proves a view is a *window*, not a copy.

---

## 5. Timing & Pacing Notes

| Part | Planned | Common Overrun | Mitigation |
|------|---------|---------------|-----------|
| Part 1: Foundation | 55 min | File path errors during INSERT delay the group | Pre-test the CSV path on your own machine; have a fallback with the full path ready to paste |
| Part 2: Constraints | 55 min | FK challenge runs long if learners hit dependency order errors | Walk through the dependency order as a class before learners write code |
| Part 3: Pipeline | 50 min | COPY file path issues eat 10+ min | Write the full path on the board/screen; require learners to confirm path before running |

---

## 6. Common Learner Questions

**Q: "Why do we need DbGate? Can't I just use the terminal?"**
A: You can — DuckDB has a CLI. DbGate provides a visual interface that makes tables, schemas, and query results easier to read while learning. In production, data engineers often use CLI tools; during learning, visual feedback helps.

**Q: "What's the difference between DuckDB and PostgreSQL?"**
A: DuckDB runs in-process (no server needed, perfect for analytics and learning). PostgreSQL runs as a server (production-grade, handles concurrent users). The SQL syntax is very similar — skills learned in DuckDB transfer directly.

**Q: "What if two people try to INSERT to the same table at the same time?"**
A: This is concurrency control — DuckDB handles it, but it's more limited than PostgreSQL. In production, databases use locking and transactions to prevent conflicts. This is a Lesson 1.5+ topic.

**Q: "What happens if I DROP the wrong table?"**
A: In DuckDB, it's gone. No Recycle Bin. No undo. In production databases, you'd restore from a backup. This is why the professional rule is: always SELECT before DROP, always have backups, always test in a dev environment first.
