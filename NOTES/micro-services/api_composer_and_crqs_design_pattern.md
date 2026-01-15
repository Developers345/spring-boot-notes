# How to Query the Data in Multiple Microservices?

If we want to **collect or aggregate data across multiple microservices**, we can achieve it mainly in **two ways**.

---

## 1. API Composer (Aggregator)
--------------------------------

- **API Composer** is a REST endpoint that **calls multiple microservices**, collects their data, **aggregates or combines it**, and then returns the final response to the **service class or end system**.
- This approach is mainly used when we need **live data**.

### Flow
```

Client
↓
API Composer
↓
Service A   Service B   Service C
↓           ↓           ↓
Aggregated Response (Live Data)

```

### Drawbacks
#### Performance Problem
- API Composer **aggregates data in JVM memory**.
- Heavy **CPU and memory utilization**.
- **Scalability issues** when the number of services or requests increases.
- Each request triggers **multiple synchronous service calls**.

👉 To overcome these problems, **CQRS** comes into the picture.

---

## 2. Command Query Responsibility Segregation (CQRS)
-----------------------------------------------------

- **CQRS** is also exposed as a REST endpoint.
- It has multiple methods to **publish events**.
- These events are consumed by different microservices.
- Data from all microservices is collected and stored in a **view-only (read-only) database**.
- Data is **queried from the read model**, not directly from microservices.

### Key Characteristics
- Data is collected **once** and stored.
- Subsequent requests are served from the **read-only database (cached data)**.
- Improves **performance and scalability**.
- Suitable for **high-read systems**.

### Flow
```

Microservices
↓ (Events)
CQRS
↓
Read-Only / View Database
↓
Client (Fast Query)

```

### Limitations
- ❌ Does **not support live data**
- ✔ Supports only **historical or eventually consistent data**

---

## Summary Comparison

| Aspect              | API Composer                 | CQRS                          |
|--------------------|------------------------------|-------------------------------|
| Data Type          | Live Data                    | Historical / Cached Data      |
| Performance        | Lower (runtime aggregation)  | High (pre-aggregated)         |
| Scalability        | Limited                      | High                          |
| Consistency        | Strong (live)                | Eventual                      |
| Use Case           | Real-time views              | Reporting, dashboards         |

---

### Conclusion
- Use **API Composer** when **real-time/live data** is mandatory.
- Use **CQRS** when **performance, scalability, and read efficiency** are more important than real-time consistency.

### Pictorial Representation - API Composer 
<img width="775" height="292" alt="API Composer" src="https://github.com/user-attachments/assets/b65556dd-e4ad-434f-9b89-e6c6dc65ee78" />

### Pictorial Representation - CRQS(Command-Query-Responsibility-Segregator)
<img width="871" height="339" alt="Command-query-responsiblity-segregator" src="https://github.com/user-attachments/assets/38377838-17ac-45f1-9ac7-2448d6fdcf4b" />

