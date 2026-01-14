# Kubernetes + Microservices + Service Discovery (Eureka) – Architecture Explanation

## 1. Kubernetes Cluster Overview
--------------------------------
- The system runs on a **Kubernetes cluster**.
- The cluster consists of:
  - **Master Node (Control Plane)**
  - **Multiple Worker Nodes**

### Master Node Responsibilities
- Manages the overall cluster.
- Responsible for:
  - Scheduling Pods on worker nodes
  - Monitoring node and pod health
  - Scaling services up/down
  - Deploying microservices
- Acts as the **Microservice Deployer**.

---

## 2. Worker Nodes
------------------
- Worker nodes are the machines where applications actually run.
- Each worker node:
  - Has limited CPU and memory (example: **16 GB RAM, 8 Core CPU**)
  - Runs **multiple Pods**
- Example shown:
  - Worker Node 1 → Pods (1, 4)
  - Worker Node 2 → Pods (2, 3)

---

## 3. Pod Concept (IMPORTANT)
-----------------------------
> **Pod = smallest deployable unit in Kubernetes**

- A **Pod** is a **logical grouping of one or more containers**.
- Containers inside a Pod:
  - Share the same **IP address**
  - Share **storage volumes**
  - Share **network namespace**
- Pods are created, destroyed, and replicated by Kubernetes.

### Why Pods?
- Some containers must run **together**
- Example:
  - Application container + sidecar (logging, config, monitoring)
- Pods ensure **co-location and shared resources**

---

## 4. Common Resources Pod
--------------------------
- A special Pod may exist for:
  - Config files
  - Certificates
  - Shared libraries
- Other Pods can access these via:
  - Volumes
  - ConfigMaps
  - Secrets

---

## 5. Microservices Deployment
-------------------------------
- Each **microservice** is deployed:
  - Independently
  - In its own **container**
  - Usually **one container per Pod**
- Microservices can be:
  - Scaled independently
  - Updated without affecting others

---

## 6. Eureka Service Discovery
-------------------------------
- **Eureka Server** runs as a **Pod inside Kubernetes**.
- It acts as a **Service Discovery Registry**.

### What Eureka Does
- Maintains a list of:
  - Available microservices
  - Their network locations (IP + Port)
- Services:
  - **Register themselves** with Eureka
  - **Send heartbeats** periodically
- If a service stops sending heartbeats:
  - Eureka removes it from the registry

---

## 7. Service Registration Flow
--------------------------------
1. Microservice starts inside a Pod
2. It registers itself with Eureka Server
3. Sends periodic **heartbeats**
4. Eureka keeps service metadata updated

---

## 8. Client-Side Load Balancing (Ribbon)
------------------------------------------
- Mobile application uses:
  - **Ribbon**
  - **Zuul**
  - **Hystrix**

### Ribbon
- Performs **client-side load balancing**
- Chooses one instance from multiple service instances
- Works using Eureka service registry

> No centralized load balancer is required

---

## 9. Zuul
-----------
- Zuul is an cilent side API sending the request to server.

---

## 10. Fault Tolerance (Hystrix)
--------------------------------
- Hystrix provides:
  - Circuit breaker pattern
  - Fallback mechanisms
- Prevents cascading failures
- If a service fails:
  - Hystrix returns fallback response
  - System remains stable

---

## 11. Why Traditional Load Balancer Is NOT Used
-------------------------------------------------
- Traditional load balancer:
  - Routes requests based on node load
- Problem in microservices:
  - Not all nodes run all services
- Solution:
  - Client-side load balancing (Ribbon)
  - Service discovery (Eureka)

---

## 12. Database Per Microservice
--------------------------------
- Each microservice:
  - Has its **own database**
  - No database sharing
- Benefits:
  - Loose coupling
  - Independent scaling
  - Independent schema evolution

---

## 13. Kubernetes + Eureka Together (Reality)
----------------------------------------------
- Kubernetes already provides:
  - Service discovery (K8s Service)
  - Load balancing
- Eureka is often used when:
  - Migrating from legacy systems
  - Using Spring Cloud Netflix stack
- In modern setups:
  - Kubernetes native Service + Ingress often replaces Eureka

---

## 14. Key Benefits of This Architecture
-----------------------------------------
- High availability
- Horizontal scalability
- Fault tolerance
- Independent deployment
- Cloud-native ready
- No single point of failure

---

## 15. Summary
---------------
- Kubernetes manages **infrastructure**
- Pods run **containers**
- Eureka manages **service discovery**
- Ribbon performs **client-side load balancing**
- Zuul handles **API Gateway**
- Hystrix provides **resilience**
- Each microservice is **independent and scalable**

## Pictorial Representation
----------------------------
<img width="1351" height="523" alt="Micro Services deployable process in kubernetes" src="https://github.com/user-attachments/assets/45991537-6ba2-4dd6-88f3-b6b427afbd58" />
