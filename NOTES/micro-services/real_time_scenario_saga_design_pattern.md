# Real-Time Scenario: Saga Design Pattern in a Bike Rental System

## Real-Time Business Scenario

### Use Case: Rent a Bike

A user wants to rent a bike using a mobile or web application.

### Microservices Involved

1. **Rental Service**
   - Manages bike availability and rentals
   - Owns **Rental Database**

2. **Payment Service**
   - Handles payments and transactions
   - Owns **Payment Database**

Each service:
- Has its **own database**
- Has **its own local transaction**
- Must remain **independently deployable**

---

## Why Not Use a Global Transaction?

If we try to use **2-Phase Commit (Global Transaction)**:

- Both databases must be locked until payment completes
- Threads wait across services
- Concurrency drops drastically
- System becomes slow under load

👉 This directly violates **microservices principles**.

---

## Saga-Based Real-Time Flow (Event-Driven)

### Step-by-Step Flow

### Step 1: User Requests Bike Rental
- User clicks **“Rent Bike”**

---

### Step 2: Rental Service (Local Transaction)

- Rental Service:
  - Creates a **PENDING** rental record
  - Commits its **local DB transaction**
  - Publishes an event: `BikeRentalCreated`

```text
Rental DB:
bike_rental_id = 101
status = PENDING
````

---

### Step 3: Payment Service Reacts to Event

* Payment Service:

  * Consumes `BikeRentalCreated` event
  * Initiates payment
  * On success:

    * Saves payment record
    * Publishes `PaymentSuccessful` event
  * On failure:

    * Publishes `PaymentFailed` event

```text
Payment DB:
payment_id = 9001
bike_rental_id = 101
status = SUCCESS
```

---

### Step 4: Rental Service Completes or Compensates

* If `PaymentSuccessful`:

  * Updates rental status → **CONFIRMED**

* If `PaymentFailed`:

  * Executes **compensating transaction**
  * Cancels rental → **CANCELLED**

```text
Rental DB:
bike_rental_id = 101
status = CONFIRMED / CANCELLED
```

✔ No global transaction
✔ No distributed locks
✔ Eventual consistency achieved

---

## Where Is the Saga?

* Each **local transaction** is part of the Saga
* Events connect the steps
* Failures are handled using **compensating actions**

---

## Saga Type Used Here

### Choreography-Based Saga

* No central coordinator
* Each service:

  * Listens to events
  * Performs its local transaction
  * Publishes the next event

---

## Pictorial Representation: Why One Service Should NOT Call Another Directly

### ❌ Direct Synchronous Communication (Anti-Pattern)

```text
User Request
    |
    v
Rental Service  ----HTTP Call---->  Payment Service
    |                                   |
    |---- DB Transaction (LOCK)         |---- DB Transaction (LOCK)
    |
Thread waits ❌  |  If Payment is slow ❌
```

### Problems

* Tight coupling
* Blocking threads
* Cascading failures
* Poor scalability

---

### ✅ Event-Driven Communication Using Kafka (Recommended)

```text
User Request
    |
    v
Rental Service
    |
(Local Transaction)
    |
Publish Event (Kafka Topic)
    |
    v
Payment Service
    |
(Local Transaction)
    |
Publish Result Event
    |
    v
Rental Service (Finalize / Compensate)
```

![Image](https://microservices.io/i/sagas/Create_Order_Saga_Orchestration.png)

![Image](https://dz2cdn1.dzone.com/storage/temp/13541021-spaghetti-microservices.png)

![Image](https://solace.com/wp-content/uploads/2019/11/Orchestration-VS-Choreography.png)

---

## Why Event-Driven Is Better

| Aspect                    | Direct Call | Event-Driven (Kafka) |
| ------------------------- | ----------- | -------------------- |
| Coupling                  | Tight       | Loose                |
| Thread Blocking           | Yes         | No                   |
| Scalability               | Low         | High                 |
| Fault Tolerance           | Poor        | Strong               |
| Microservice Independence | ❌           | ✔                    |

---

## Summary

* ❌ Global transactions do not scale in microservices
* ❌ Direct service-to-service calls create tight coupling
* ✔ Saga pattern ensures **eventual consistency**
* ✔ Kafka enables **non-blocking, asynchronous communication**
* ✔ Compensating transactions handle failures cleanly

### Real-World Rule

> **In microservices, never share databases, never use global transactions, and never block one service waiting for another.**

**Saga Pattern + Event-Driven Architecture = Production-Grade Microservices**

