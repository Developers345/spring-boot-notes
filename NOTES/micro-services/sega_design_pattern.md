# Data Consistency and Transactionality in Microservices

If we break a system / application into multiple independent microservices, a key challenge is **how to achieve data consistency and transactionality** across services.

---

## Problem Statement

Let’s consider a **bike rental system**.

- A user wants to rent a bike.
- A bike should be allotted **only after successful payment**.
- These two processes belong to **two different services**:
  - **Rental Service** → Rental database
  - **Payment Service** → Payment database

Since each service owns its own database, maintaining **data consistency** becomes complex.

---

## Example Scenario

### Rental Table (Rental Database)

**bike_rental**

| bike_rental_id | user_id | bike_id | rented_date | rented_time | location | charge_per_hour | coupon_applied |
|----------------|---------|---------|-------------|-------------|----------|------------------|----------------|

---

### Payment Table (Payment Database)

**payment**

| payment_id | user_id | payment_method | amount_paid | bike_rental_id |
|------------|---------|----------------|-------------|----------------|

---

### Expected Behavior

- When payment is successful:
  - A record should be inserted into **bike_rental**
  - A record should be inserted into **payment**
- **Both inserts must be committed together**, or neither should be committed.

---

## Global Transactions (2-Phase Commit)

To achieve atomicity across multiple databases, we can use:

- **2-Phase Commit (2PC)**
- **Global Transactions**

### Drawbacks of Global Transactions

- ❌ **Low concurrency**
- ❌ **High latency**
- ❌ **Database locks are held for a long time**
- ❌ **Poor scalability**

### Why Global Transactions Are an Anti-Pattern in Microservices

- Microservices aim for **independent scalability and high concurrency**
- Global transactions:
  - Lock database resources across services
  - Force services to wait for each other
  - Increase thread blocking time
  - Degrade overall system performance

👉 Hence, **global transactions kill the core motivation of microservices**.

---

## Saga Design Pattern

To overcome the problems of global transactions, microservices use the **Saga Design Pattern**.

### What Is a Saga?

- A **Saga** is a sequence of **local transactions**
- Each service:
  - Executes its **own local transaction**
  - Publishes an **event** for the next step
- If a step fails:
  - **Compensating transactions** are executed to undo previous changes

---

### Key Characteristics of Saga

- ✔ No global transaction
- ✔ No distributed database locks
- ✔ Event-driven consistency
- ✔ High scalability and concurrency

---

### Saga Flow (Bike Rental Example)

1. **Rental Service**
   - Creates a rental entry (local transaction)
   - Publishes `BikeRentedEvent`
2. **Payment Service**
   - Processes payment (local transaction)
   - Publishes `PaymentCompletedEvent`
3. **Failure Case**
   - If payment fails:
     - A **compensating transaction** cancels the rental

---

## Why Services Should Not Communicate Directly

### 1. Tight Coupling

- Direct synchronous communication leads to **tight coupling**
- Independent development and deployment become difficult
- Microservices should be **loosely coupled**

👉 Solution: Use **message brokers** (JMS / Kafka)

---

### 2. Resource Blocking

- Direct service-to-service calls are **blocking**
- Calling service thread waits until the called service completes
- Increases:
  - CPU usage
  - Memory consumption
  - Thread contention

---

### Why Kafka Over JMS?

- ❌ **JMS** is heavyweight (requires Java EE container)
- ✔ **Kafka** is:
  - Lightweight
  - Highly scalable
  - Event-driven
  - Ideal for microservices

---

## Saga Implementation Approaches

Saga can be implemented in **two ways**:

---

## 1. Choreography-Based Saga

- No central coordinator
- Each service:
  - Executes a local transaction
  - Publishes an event
  - Next service reacts to the event

### Characteristics

- ✔ Simple
- ✔ Decentralized
- ❌ Harder to track flow
- ❌ Complex debugging for large systems

---

## 2. Orchestration-Based Saga

- A central **Saga Orchestrator** controls the flow
- Orchestrator:
  - Tells each service which local transaction to execute
  - Decides compensating actions on failure

### Characteristics

- ✔ Clear workflow
- ✔ Better visibility
- ✔ Easier error handling
- ❌ Slightly more complex setup

---

## Summary

- ❌ Global transactions are not suitable for microservices
- ✔ Saga pattern ensures **eventual consistency**
- ✔ Each service owns its database and transaction
- ✔ Kafka-based event-driven communication ensures loose coupling
- ✔ Saga supports high concurrency and scalability

**Saga Pattern = Preferred approach for data consistency in microservices**

## Pictorial representation - basic communication between microservices
<img width="539" height="305" alt="basic communication between micro-servies" src="https://github.com/user-attachments/assets/9433594f-779d-4196-9ed6-4aba3c183425" />

## Pictorial representation - why one service not call another service directly 
<img width="753" height="436" alt="why one service not call another service directly" src="https://github.com/user-attachments/assets/86f89d9e-87ea-482c-9902-a9a7259d557a" />

## Pictorial representation - sega design pattern for data consistency 
<img width="920" height="466" alt="sega design pattern for data consistency" src="https://github.com/user-attachments/assets/115acffb-14cc-4796-9cd3-42f2d0776224" />
