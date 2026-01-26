# No-SQL Databases / Semi-Structured Databases

There are limitations in **Relational Database Management Systems (RDBMS)** when catering to all types of applications. Earlier, due to the absence of alternatives, RDBMS were used for every kind of application, which introduced several pain points.

## Problems with RDBMS

### 1. Memory Wastage When Schema Is Not Fixed
When different records contain different fields, we must create a table with all possible columns.  
Since every record does not have values for all columns, many columns store `NULL` values, resulting in **huge memory wastage**.

### 2. Scalability Is Difficult to Achieve
Join-based queries in RDBMS consume a large amount of **CPU and memory**.  
To handle such workloads, databases must run on **high-grade hardware**, which significantly increases the cost of scaling.

### 3. Limited Concurrency
RDBMS are **OLTP (Online Transaction Processing)** systems.  
Due to transaction isolation levels and locking mechanisms, achieving **high concurrency** is difficult.

Even when an application:
- Does not require transaction processing, and
- Does not have relational data requirements,

we still had to use RDBMS earlier, leading to **unnecessary cost** for scalability and concurrency.

---

## Emergence of Alternate Database Systems

To overcome these problems, alternate database management systems emerged:

1. **No-SQL or Semi-Structured Databases**
2. **Object Storage Databases / Un-Structured Databases**

---

## No-SQL / Semi-Structured Databases

The way data is stored and represented varies from one No-SQL database product to another. Let us generalize the concept.

### Key Characteristics

- Tables do **not** have a fixed schema.
- While inserting data, users store information as **key/value pairs**.
- Each record (row/document) may contain a **different set of fields**.
- Because there is no fixed structure, these databases are called **semi-structured databases**.
- Since tables and columns are not predefined, **SQL cannot be used** for querying data in the traditional way. Hence, they are called **No-SQL databases**.

### Suitable Use Cases

No-SQL databases are well suited for:
1. Key/value data where each record has **varied fields**
2. Data with **no relational dependencies**
3. Applications that **do not require transaction-based concurrency**

---

## Popular No-SQL Databases

Some widely used No-SQL database management systems are:

1. MongoDB  
2. Cassandra  
3. Oracle Big Data  
4. CouchDB  
5. Graph Databases  
6. Firebase  
7. AWS DynamoDB  

---

## Storage Models in No-SQL Databases

Different No-SQL databases use different storage models:

1. **MongoDB / CouchDB**
   - Uses **JSON document format**
   - One record is a **document**
   - Documents are stored in **collections**
   - Each document can have a different set of key/value pairs

2. **Cassandra / DynamoDB**
   - Pure **key/value-based storage**
