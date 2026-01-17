# Ribbon
--------

→ **Netflix Ribbon** provides a **client-side load balancer** that distributes client traffic across multiple instances of a service.

---

## Role of Eureka vs Ribbon
---------------------------

→ **Eureka Server** is a **service discovery** component only.

→ Eureka **does NOT act as a load balancer**, which means:
- It does **not** track which client requests are routed to which service nodes
- It does **not** route requests based on load or traffic on service instances

---

## Why Load Balancer is Required
-------------------------------

→ Even if a service is **scaled up to multiple instances**, without a load balancer:
- Client traffic may always go to **only one instance**
- Other instances remain underutilized
- Scaling becomes ineffective

---

## Netflix Ribbon – Client-Side Load Balancer
--------------------------------------------

→ Since server-side load-based routing is not available in this setup,
Netflix introduced a **client-side load balancer** called **Ribbon**.

→ Ribbon works at the **client side**, not on the server.

---

## How Ribbon Works
------------------

→ Ribbon:
- Keeps track of requests sent from the client
- Knows all available instances of a service
- Distributes traffic across service instances

→ Default load-balancing strategy:
- **Round-robin algorithm**

---

## Ways to Use Ribbon
--------------------

### 1. Ribbon with Eureka (Service Discovery Based)
- Ribbon discovers service instances from **Eureka Server**
- Automatically distributes requests across all instances
- Uses round-robin load balancing

**Flow:**
```

Client → Eureka → Ribbon → Service Instances

````

---

### 2. Ribbon without Eureka (Static Server List)
- Manually provide the list of service instances
- No service discovery required
- Ribbon routes traffic using the provided server list

Example:
```text
listOfServers = host1:8080, host2:8080, host3:8080
````

→ In this approach:

* Eureka client is **not required**
* Ribbon directly communicates with known service instances
* Useful for simple or static environments

---

## Key Points

---

* Ribbon is a **client-side load balancer**
* Works independently of server-side load balancers
* Improves utilization of multiple service instances
* Commonly used with:

  * Eureka (dynamic discovery)
  * Static server lists (manual configuration)

---
