# Delta Lake Concepts in Databricks

## Overview

Delta Lake is an open-source storage layer that brings **ACID transactions**, **schema enforcement**, **data versioning**, and **reliable data processing** to data lakes. In Databricks, Delta Lake is the default storage format used for building scalable and reliable data pipelines.

This repository demonstrates the core Delta Lake concepts including:

* Delta Lake Architecture
* Delta Tables
* Delta Log (`_delta_log`)
* INSERT Operations
* UPDATE Operations
* DELETE Operations
* Time Travel
* Transaction Management

---

## What is Delta Lake?

Delta Lake is a storage framework built on top of existing cloud storage systems such as:

* Azure Data Lake Storage (ADLS)
* Amazon S3
* Google Cloud Storage

It extends Parquet files with transaction logs and metadata management to provide:

* ACID Transactions
* Data Reliability
* Data Versioning
* Schema Enforcement
* Schema Evolution
* Time Travel
* Unified Batch and Streaming Processing

---

## Delta Table

A Delta Table is a collection of data files stored in Parquet format along with transaction logs maintained in the `_delta_log` directory.

### Creating a Delta Table

```sql
CREATE TABLE employee (
    emp_id INT,
    emp_name STRING,
    salary DOUBLE,
    department STRING
)
USING DELTA;
```

### Insert Sample Data

```sql
INSERT INTO employee VALUES
(101,'John',50000,'IT'),
(102,'David',60000,'HR'),
(103,'Smith',70000,'Finance');
```

---

## Delta Log (`_delta_log`)

Every Delta table contains a hidden folder called:

```text
_delta_log
```

This folder stores transaction logs in JSON and checkpoint Parquet files.

### Purpose of Delta Log

* Tracks all table changes
* Maintains transaction history
* Supports rollback operations
* Enables Time Travel
* Provides ACID compliance

### Example Structure

```text
employee/
│
├── part-00000.parquet
├── part-00001.parquet
│
└── _delta_log/
    ├── 00000000000000000000.json
    ├── 00000000000000000001.json
    ├── 00000000000000000002.json
    └── checkpoint.parquet
```

Each transaction creates a new version in the Delta Log.

---

## INSERT Operation

Insert new records into a Delta Table.

### SQL Example

```sql
INSERT INTO employee
VALUES (104,'Michael',80000,'IT');
```

### PySpark Example

```python
spark.sql("""
INSERT INTO employee
VALUES (104,'Michael',80000,'IT')
""")
```

### Benefits

* ACID compliant
* Reliable writes
* Transaction tracking

---

## UPDATE Operation

Update existing records in a Delta Table.

### SQL Example

```sql
UPDATE employee
SET salary = 90000
WHERE emp_id = 101;
```

### PySpark Example

```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forName(spark, "employee")

delta_table.update(
    condition="emp_id = 101",
    set={"salary": "90000"}
)
```

### Benefits

* Supports row-level updates
* Maintains transaction history
* Enables rollback through Time Travel

---

## DELETE Operation

Delete records from a Delta Table.

### SQL Example

```sql
DELETE FROM employee
WHERE emp_id = 102;
```

### PySpark Example

```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forName(spark, "employee")

delta_table.delete("emp_id = 102")
```

### Benefits

* Efficient row-level deletion
* Fully transactional
* Historical versions preserved

---

## Viewing Table History

Delta Lake maintains the history of all operations performed on a table.

### SQL

```sql
DESCRIBE HISTORY employee;
```

### Output Includes

* Version Number
* Timestamp
* User Information
* Operation Type
* Operation Metrics

Example:

| Version | Operation    |
| ------- | ------------ |
| 0       | CREATE TABLE |
| 1       | INSERT       |
| 2       | UPDATE       |
| 3       | DELETE       |

---

## Time Travel

Time Travel allows querying previous versions of a Delta Table.

### Query by Version

```sql
SELECT *
FROM employee VERSION AS OF 1;
```

### Query by Timestamp

```sql
SELECT *
FROM employee TIMESTAMP AS OF '2026-08-01';
```

### PySpark Example

```python
df = spark.read.format("delta") \
    .option("versionAsOf", 1) \
    .table("employee")
```

### Use Cases

* Data Recovery
* Audit Requirements
* Historical Analysis
* Debugging Data Pipelines

---

## ACID Transactions

Delta Lake provides complete ACID transaction support.

### Atomicity

All operations succeed or fail completely.

### Consistency

Data remains valid before and after transactions.

### Isolation

Concurrent operations do not interfere with each other.

### Durability

Committed data is permanently stored.

---

## Advantages of Delta Lake

### Reliability

* ACID transactions
* Data consistency
* Fault tolerance

### Performance

* Data skipping
* File compaction
* Optimized queries

### Governance

* Audit history
* Schema enforcement
* Time Travel support

### Scalability

* Handles petabytes of data
* Supports batch and streaming workloads

---

## Common Commands

### Describe History

```sql
DESCRIBE HISTORY employee;
```

### View Current Data

```sql
SELECT * FROM employee;
```

### View Older Version

```sql
SELECT * FROM employee VERSION AS OF 1;
```

### Restore Table

```sql
RESTORE TABLE employee TO VERSION AS OF 1;
```

### Optimize Table

```sql
OPTIMIZE employee;
```

### Vacuum Old Files

```sql
VACUUM employee RETAIN 168 HOURS;
```

---

## Delta Lake Workflow

```text
Raw Data
    │
    ▼
Delta Table
    │
    ├── INSERT
    ├── UPDATE
    ├── DELETE
    │
    ▼
_delta_log
(Transaction History)
    │
    ▼
Time Travel & Recovery
```

---

## Conclusion

Delta Lake enhances traditional data lakes by providing ACID transactions, transaction logging, schema enforcement, and Time Travel capabilities. The `_delta_log` directory acts as the backbone of Delta Lake, tracking every change and enabling reliable data management in Databricks.

Using Delta Tables, organizations can build scalable, secure, and high-performance data engineering solutions while maintaining complete historical visibility of data changes.

