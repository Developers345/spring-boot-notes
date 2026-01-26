# Relational Databases vs NoSQL (Semi-Structured Databases) – Explained Simply

## 1. Relational Database Management System (RDBMS)

### What it is
- Designed to **store and process structured business data**
- Data is stored in **tables**
  - **Columns** → fixed schema (structure)
  - **Rows** → actual records (each row has the same number of columns)

### Common RDBMS Software
- MySQL
- Oracle
- PostgreSQL
- SQL Server
- MariaDB

### Key Characteristics
- Schema is **predefined and fixed**
- Strong **ACID transactions**
- Uses **primary keys and foreign keys** to model relationships

### Advantages
1. **Scalability (clustering)** – limited but supported
2. **High availability**
3. **Security** – username/password authentication
4. **Transactional consistency** (ACID)
5. **Relationships** across tables using joins

---

## 2. Limitations of RDBMS (Why NoSQL Came In)

NoSQL / semi-structured databases were introduced to overcome some major limitations of RDBMS.

---

## 3. Problem 1: Memory Wastage (Fixed Schema)

### Issue
- Every row must have **all columns**, even if some values are not applicable.
- Missing values are stored as **NULL**, which wastes memory.

### Example (Product Table)
```

product_no | product_nm | description | manufacturer | color | expiry_dt | price | warranty | batch_no | category | weight | height | width

```

Many rows may not need:
- expiry date
- warranty
- dimensions (height/width)

➡ Still, **NULL values must be stored**

### Impact
- Huge **storage overhead**
- Inefficient for **flexible or evolving data models**

---

## 4. Problem 2: Costly Relationships (Joins)

### Scenario
- `customer` table → 1,000 records
- `product` table → 10,000 records
- Relationship via `customer_no` (foreign key)

### Query Requirement
> “Give me every customer and the products they bought”

### What Happens Internally
- For each customer record:
  - Database scans product table to find matching `customer_no`
- Rough comparison count:
```

1000 customers × 10000 products = 10,000,000 comparisons

```

### Impact
- **High CPU usage**
- **High memory consumption**
- Performance degrades as data grows
- Difficult to scale horizontally

---

## 5. Problem 3: Transactions & Locking (Scalability Issue)

### Real-Time Example: Train Ticket Reservation

#### Steps
1. Query available seats
2. Lock table/rows
3. Reserve ticket
4. Unlock table/rows

### Why Locking Is Needed
- To avoid **data inconsistency**
- Prevent two users from booking the same seat

### Problem
- While one transaction holds a lock:
  - Others must **wait**
- Even if:
  - Plenty of CPU
  - Plenty of memory

### High Traffic Scenario
- 1 lakh users trying to book tickets
- Only **one transaction at a time** modifies the table

### Impact
- Severe **performance bottleneck**
- Poor **horizontal scalability**
- RDBMS works best for **OLTP**, not massive concurrent access

---

## 6. Summary of RDBMS Problems

1. **Memory wastage**
   - Fixed schema + NULL values
2. **High CPU & memory usage**
   - Due to joins and relationships
3. **Scalability challenges**
   - Caused by locking and strict transactions

---

## 7. Why People Still Used RDBMS Earlier

- No good alternative existed
- Used even when applications needed:
  - Flexible fields
  - No relationships
  - Less transactional guarantees

---

## 8. Why NoSQL Databases Emerged

NoSQL databases solve these problems by providing:

- **Schema flexibility** (no fixed columns)
- **Denormalized data** (fewer joins)
- **Better horizontal scalability**
- Designed for **high-volume, high-velocity data**

### Best Fit Use Cases
- Semi-structured data
- Large-scale distributed systems
- High concurrency applications
- When relationships & strict transactions are not mandatory

---

## 9. Final Thought

- **RDBMS** → Best for strong consistency, structured data, and complex transactions  
- **NoSQL** → Best for scalability, flexibility, and high-performance distributed systems  

👉 Choice depends on **application requirements**, not trends.
```
