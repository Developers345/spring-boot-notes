# Legacy Web Application Using EJB in J2EE Servers
------------------------------------------------

## Overview

- **EJB (Enterprise JavaBeans)** is a distributed component model developed using Java.
- It is used to build **enterprise-level, server-side components** such as:
  - Business logic
  - Transaction management
  - Persistence logic

---

## Application Architecture

### 1. Web Tier
- Developed using:
  - JSP / HTML
  - Servlets
  - Action frameworks (Struts)
  - Controllers (Spring MVC)
- Characteristics:
  - Acts as the **presentation layer**
  - Handles user requests and responses
  - Independent component deployed on the J2EE server

---

### 2. Service & Persistence Tier
- Developed using:
  - EJBs for business logic
  - DAOs for data access
- Characteristics:
  - Persistence tier is an **independent component**
  - EJBs manage:
    - Transactions
    - Security
    - Persistence
    - Distributed communication

---

## Session Management and Scalability Issues

- The **J2EE server framework**:
  - Maintains user session objects
  - Handles session replication across the cluster
- Problems:
  - Heavy server-side session management
  - Session replication increases memory and network overhead
  - Makes horizontal scalability difficult
  - Scaling the web application becomes complex and inefficient

---

## Limitations with Modern Frontend Frameworks

- Angular or React **cannot be easily used** for the web tier in this architecture because:
  - Web tier directly calls EJBs using **JNDI registry**
  - EJBs return large, complex Java object graphs
- Modern frontends expect:
  - Lightweight REST/JSON APIs
  - Stateless communication
- Tight coupling between:
  - Web tier and EJB layer
  - Server-side UI and business logic

---

## Summary
---------

- Legacy EJB-based J2EE applications:
  - Are tightly coupled
  - Rely heavily on server-side session management
  - Are difficult to scale
  - Are not well-suited for modern SPA frameworks like Angular or React
- These limitations paved the way for:
  - RESTful architectures
  - Microservices
  - Stateless web applications

## Pictorial representation - legacy enterprise applications
-------------------------------------------------------------
<img width="591" height="217" alt="legacy web application using ejb and persistence tier(dao)" src="https://github.com/user-attachments/assets/daf3bfbc-7737-478b-8751-d4115163a64c" />

# Why Spring Framework Is Not Used to Invoke Microservices or an API Gateway?
---------------------------------------------------------------------------

## Server-Side Session Management Problem

- Traditional server-side frameworks (such as **Struts** or **Spring MVC**) maintain:
  - User session objects
  - Session replication across clustered servers
- Problems caused:
  - High memory consumption
  - Network overhead due to session replication
  - Difficulty in scaling web applications horizontally

---

## Industry Solution: Client-Side Web Applications

- To overcome these issues, software industries adopted:
  - **Angular / React** as client-side web applications
  - Simple web pages capable of sending **AJAX requests**
- These client-side applications replace:
  - Server-side web frameworks (Struts, Spring MVC) at the web tier

---

## Deployment Model

- Angular applications are deployed on a **Node.js server**.
- The Node server serves:
  - Static UI assets (HTML, CSS, JavaScript)

---

## Single Page Application (SPA) Concept

- Angular is a **Single Page Application (SPA)**:
  - When a user accesses the web application for the first time:
    - The Angular application is downloaded from the Node server to the browser.
  - For subsequent interactions:
    - Angular sends requests only for **dynamic data** to:
      - Microservices
      - API Gateway

---

## Scalability and Session Management

- There is **no server-side session management**:
  - Applications are stateless
  - Each request is independent
- Benefits:
  - Easy horizontal scalability
  - No session replication overhead
- The **MVC pattern** is handled entirely on the client side:
  - Model
  - View
  - Controller

---

## Summary
---------

- Spring MVC is not ideal as a web tier for microservices-based systems.
- Client-side SPAs (Angular/React):
  - Eliminate server-side session management
  - Improve scalability
  - Enable clean separation between UI and backend services
- Backend microservices focus only on:
  - Business logic
  - Data processing

## Pictorial representation 
----------------------------
<img width="837" height="268" alt="how the angular resloves the problem with server-side frameworks" src="https://github.com/user-attachments/assets/6534f21a-8573-4d88-88dc-127c6ba834f3" />
