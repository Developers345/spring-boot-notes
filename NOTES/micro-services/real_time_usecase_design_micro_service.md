# Real-Time Use Case: Designing Microservices  
## Example – Bike Rental Application

This example shows **how to identify and design microservices** using the principles explained earlier  
(**SRP, CCP, Business Capability, and DDD**).

---

## 1. Business Problem (Real-World Scenario)

We are building a **Bike Rental Platform** similar to Rapido / Bounce.

### End-User Capabilities
- Users can register and log in
- Users can search for bikes
- Users can rent a bike
- Users can make payments
- Users can view rental history and invoices

---

## 2. Step-1: Identify Business Capabilities

From the requirements, identify **what the business does** (not technical layers).

| Business Capability | Description |
|--------------------|------------|
| User Management | Register, login, profile management |
| Bike Management | Add, update, track bikes |
| Rental Management | Book, start, end rentals |
| Payment Management | Process payments, refunds |
| Account & Billing | Statements, invoices, history |

👉 Each business capability is a **candidate microservice**.

---

## 3. Step-2: Apply Single Responsibility Principle (SRP)

Each microservice must have **only one responsibility**.

### ❌ Wrong Design
One big service:
```

BikeRentalService
├── userRegistration()
├── addBike()
├── rentBike()
├── processPayment()
├── generateInvoice()

```

❌ Violates SRP  
❌ Any change affects the entire system

---

### ✅ Correct SRP-Based Design

Each service handles **one responsibility only**.

```

UserService        → User-related logic only
BikeService        → Bike-related logic only
RentalService     → Rental lifecycle only
PaymentService    → Payment processing only
BillingService    → Invoices & statements only

```

---

## 4. Step-3: Apply Common Closure Principle (CCP)

> **Classes that change together should be packaged together**

### Example: Rental Change Request
Client says:
> “We want to charge extra if a bike is returned late”

### Impacted Area
- Rental duration
- Late fee calculation

### Where Should the Change Go?
✅ **Only inside RentalService**

```

rental-service
├── RentalController
├── RentalService
├── RentalRepository
├── LateFeeCalculator

```

❌ No changes in:
- UserService
- BikeService
- PaymentService

✔ Change is **closed within one microservice**

---

## 5. Step-4: Decomposition by Business Capability (Final Services)

### 1️⃣ User Service
**Responsibility**
- User registration
- Authentication
- Profile management

**Owns**
- User database

**Example APIs**
```

POST   /users/register
POST   /users/login
GET    /users/{id}

```

---

### 2️⃣ Bike Service
**Responsibility**
- Bike onboarding
- Availability tracking

**Owns**
- Bike database

**Example APIs**
```

POST   /bikes
GET    /bikes/available
PUT    /bikes/{id}/status

```

---

### 3️⃣ Rental Service
**Responsibility**
- Start rental
- End rental
- Calculate duration & charges

**Owns**
- Rental database

**Example APIs**
```

POST   /rentals/start
POST   /rentals/end
GET    /rentals/user/{userId}

```

---

### 4️⃣ Payment Service
**Responsibility**
- Payment processing
- Refunds
- Payment status

**Owns**
- Payment database

**Example APIs**
```

POST   /payments/pay
POST   /payments/refund
GET    /payments/{rentalId}

```

---

### 5️⃣ Billing Service
**Responsibility**
- Invoice generation
- Statements

**Owns**
- Billing database

**Example APIs**
```

GET   /billing/invoice/{rentalId}
GET   /billing/statements/{userId}

```

---

## 6. Step-5: Decomposition Using Domain-Driven Design (DDD)

### Domain: Bike Rental System

### Sub-Domains
| Sub-Domain | Microservice |
|-----------|-------------|
| Identity | User Service |
| Fleet | Bike Service |
| Rentals | Rental Service |
| Payments | Payment Service |
| Finance | Billing Service |

Each sub-domain:
- Has its **own model**
- Owns its **own database**
- Can be developed & deployed independently

---

## 7. Service Interaction (Real-Time Flow)

### Rent a Bike Flow
```

User → Rental Service
├─ checks user via User Service
├─ checks bike availability via Bike Service
├─ creates rental
└─ triggers Payment Service

```

📌 Services communicate via:
- REST APIs
- Async events (Kafka / RabbitMQ)

---

## 8. Final Microservice Architecture (Conceptual)

```

[ User Service ]        [ Bike Service ]
│                    │
└──────┐        ┌────┘
▼        ▼
[ Rental Service ]
│
▼
[ Payment Service ]
│
▼
[ Billing Service ]

```

---

## 9. Benefits Achieved

✔ Loosely Coupled  
✔ Independently Deployable  
✔ Scalable per business capability  
✔ Easy to change & maintain  
✔ Team ownership per service  

---

## 10. Key Takeaway

To design real-time microservices:
1. Identify **business capabilities**
2. Apply **SRP** (one responsibility per service)
3. Apply **CCP** (changes stay within service)
4. Align services with **DDD sub-domains**
5. Ensure **independent deployment & data ownership**

This is how **real-world microservices are designed in production systems**.
```
