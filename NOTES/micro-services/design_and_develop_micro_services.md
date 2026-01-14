# How to Design / Develop Microservices?

## Problem Statement
We have system requirements that should be built into an end system / application.  
If we want to develop this application based on **microservice architecture**:

- How do we break the system into services?
- How many services should we develop?
- In short, **how do we identify a microservice while developing the system?**

---

## General Guidelines for Designing Classes as Services

### 1. Single Responsibility Principle (SRP)
A class should perform **only one responsibility**, and there should be **only one reason for change**.

**In other words:**
- A class should perform only one business responsibility.
- If a change exists, it should be for **one and only one reason**.

**Example:**
- A `ProductService` class should contain **only product-related business functionality**.
- Any change in this class should be related **only to product requirements**.

This ensures:
- Clear ownership
- Easier maintenance
- Reduced impact of changes

---

### 2. Common Closure Principle (CCP)
Another Object-Oriented Design principle which states:

> **Classes that change together should be packaged together.**

**In simple terms:**
- If a change request comes from the client, the change should be confined **within classes of the same package**.
- This principle is known as the **Common Closure Principle (CCP)**.

**Why adopt SRP and CCP?**
- To minimize collaboration with other teams during changes
- To ensure changes are implemented and deployed independently
- To reduce ripple effects across services

**Outcome:**
- Loosely coupled services
- Independently deployable microservices

---

## Ways to Identify or Break a System into Microservices

### 1. Decomposition by Business Capability / Responsibility
Design or break the system into microservices based on **business capabilities**.

**Business Capability:**
- An operation carried out by the business to meet end-user requirements.
- Represents *what the business does*, not *how it is implemented*.

**Example: Bike Rental System**

Business capabilities (actionable operations performed by end users):

1. User Management  
2. Rentals  
3. Bike Management  
4. Account & Statements  
5. Payments  

Each capability can be designed as an **independent microservice**.

---

### 2. Decomposition by Domain-Driven Design (DDD)
A project or application belongs to **one domain**, but within it, we can identify multiple **sub-domains**.

**Example:**
- Domain: Bike Rental System  
- Sub-domain: Automobiles

**Sub-domains in a Bike Rental System:**
1. User Management  
2. Rentals  
3. Bike Management  
4. Accounts and Payments  

Within each business grouping, the **related functionalities together form a sub-domain**.

---

## Key Takeaway
- Apply **SRP** and **CCP** while designing services
- Decompose systems based on:
  - Business capabilities **or**
  - Domain-driven sub-domains
- This approach helps in building:
  - Loosely coupled systems
  - Independently deployable microservices
  - Scalable and maintainable architectures
```
