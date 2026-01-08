# Lab 2: SQL DML (Data Manipulation Language)

## Overview
This lab covers **SQL DML (Data Manipulation Language)** operations, which are fundamental SQL commands used to retrieve, insert, update, and delete data from database tables. This lab is based on the SQL DML concepts from the [COS3103 Database Course](https://luckkrit.github.io/cos3103/slides/2_68/sql_dml).

## What is SQL DML?

SQL DML refers to commands that manipulate data in a database. Key DML operations include:

- **SELECT** - Retrieve data from tables
- **INSERT** - Add new rows to tables
- **UPDATE** - Modify existing data
- **DELETE** - Remove rows from tables

## Database Setup

### Prerequisites
Before starting this lab, you need to have:
1. **PostgreSQL** installed ([Download PostgreSQL](https://www.postgresql.org/download/))
2. **PostGIS Extension** (Optional for spatial data)

### Setting Up the Database

#### 1. Enable PostGIS Extension
```sql
CREATE EXTENSION postgis;
```

#### 2. Load Classic Models Schema
Download and execute the classic models database schema:
```sql
-- Download from: https://luckkrit.github.io/cos3103/sql/postgresql-classicmodels.sql
SET search_path TO public, classicmodels;
```

#### 3. Set Search Path
```sql
SET search_path TO public, classicmodels;
```

## Database Schema Overview

The **Classic Models** database contains 8 main tables:

| Table | Purpose |
|-------|---------|
| **Customers** | Stores customer information |
| **Products** | Stores product/car model details |
| **ProductLines** | Contains product line categories |
| **Orders** | Stores sales orders |
| **OrderDetails** | Contains individual line items per order |
| **Payments** | Records customer payments |
| **Employees** | Stores employee and organizational structure |
| **Offices** | Stores sales office locations |

---

## SQL Query Fundamentals

### Query Execution Order (Important!)

SQL queries are processed in this order:

1. **FROM** - Identify the table(s)
2. **JOIN** - Combine tables
3. **WHERE** - Filter rows
4. **GROUP BY** - Organize data into groups
5. **HAVING** - Filter groups
6. **SELECT** - Choose columns
7. **ORDER BY** - Sort results

--

## References

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [SQL String Functions](https://www.postgresql.org/docs/18/functions-string.html)
- [SQL Official Tutorial](https://www.w3schools.com/sql/)
- [COS3103 Database Course Slides](https://luckkrit.github.io/cos3103/slides/2_68/sql_dml)

---

## Database Download Links

- **Classic Models Schema:** [postgresql-classicmodels.sql](https://luckkrit.github.io/cos3103/sql/postgresql-classicmodels.sql)
- **PostgreSQL:** [PostgreSQL Download](https://www.postgresql.org/download/)