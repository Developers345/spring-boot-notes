# Containerized Microservices Architecture – Explained

## 1. What is a Container?
- A **container** is an application packaged along with:
  - Its runtime
  - Required libraries
  - Dependencies
  - Configuration
- Containers run in **isolated environments**, even when multiple containers run on the same machine.
- This isolation avoids dependency conflicts between applications.

### Key Benefits
- Lightweight compared to virtual machines
- Faster startup time
- Portable across environments (Dev / QA / Prod)
- Consistent behavior everywhere

---

## 2. Docker and Docker Images
- **Docker** is a containerization platform.
- An **image** is a blueprint (read-only template) used to create containers.

### Docker Image Contains
- Application code
- Runtime (JDK, Node, etc.)
- OS-level dependencies
- Application configuration

### Docker Container
- A running instance of a Docker image.

---

## 3. Docker Registry
- A **Docker Registry** stores Docker images.
- Examples:
  - Docker Hub
  - AWS ECR
  - Azure Container Registry
  - Private registry

### Flow
1. Build Docker image
2. Push image to registry
3. Pull image from registry during deployment

---

## 4. Kubernetes (K8s)
- **Kubernetes** is a **container orchestration platform**.
- It manages:
  - Deployment
  - Scaling
  - Networking
  - Self-healing
  - Load balancing of containers

### Kubernetes Responsibilities
- Start containers
- Restart failed containers
- Scale containers up/down
- Manage service networking
- Perform rolling updates

---

## 5. Kubernetes Manifest Files
- A **manifest** is a YAML file that describes:
  - What to deploy
  - How many replicas
  - Which image to use
  - Networking rules

### Common Manifest Types
- Deployment
- Service
- ConfigMap
- Secret
- Ingress

---

## 6. Cluster Management
Kubernetes replaces traditional **JEE container responsibilities**.

### Cluster Management Includes
- Adding managed servers (nodes)
- Removing managed servers
- Monitoring node health
- Auto-healing failed pods
- Auto-scaling applications

---

## 7. Application Deployment Rollouts
- Kubernetes supports:
  - Rolling updates
  - Zero-downtime deployments
  - Rollback to previous versions

### Example
- Deploy v2 while v1 is still serving traffic
- Gradually shift traffic to v2
- Roll back instantly if issues occur

---

## 8. Monitoring (Prometheus)
- **Prometheus** is used for:
  - Metrics collection
  - Health monitoring
  - Performance tracking

### Typical Metrics
- CPU usage
- Memory usage
- Request count
- Error rates
- Pod health

---

## 9. Service Discovery (Eureka)
- **Eureka** acts as a **Service Registry**.
- Each microservice:
  - Registers itself with Eureka
  - Discovers other services dynamically

### Why Service Discovery?
- Services scale dynamically
- IP addresses change frequently
- Hardcoding service locations is impossible

---

## 10. Stateless REST Microservices
- All services are **REST-based** and **stateless**.
- Stateless means:
  - No session data stored on server
  - Every request contains complete information

### Advantages
- Easy scaling
- No session replication required
- Better fault tolerance

---

## 11. Session Management & Fault Tolerance
- Since services are stateless:
  - No HTTP session replication required
  - Any instance can serve any request
- This naturally supports **High Availability (HA)**

---

## 12. Load Balancing – Why Traditional LB Is Not Enough
### Traditional Load Balancer
- Routes requests based on:
  - Server-level load
- Works well in monolithic or JEE environments

### Problem in Microservices
- Each service runs independently
- Services are deployed on different nodes
- Load balancer does NOT know:
  - Which service runs on which node

---

## 13. How Requests Are Routed in Microservices
- Service discovery (Eureka)
- Client-side load balancing (e.g., Spring Cloud LoadBalancer)
- Kubernetes Services handle pod-level routing

### Flow
1. Client calls Service A
2. Service A queries Eureka
3. Eureka returns available instances
4. Client-side load balancer selects one instance
5. Request is routed directly

---

## 14. Web Servers in the Diagram
- Each **web server** represents:
  - An independent microservice instance
  - Running inside its own container
- Services scale horizontally by adding more containers

---

## 15. Summary Architecture Flow
1. Application packaged as Docker image
2. Image stored in Docker Registry
3. Kubernetes pulls image
4. Containers are created and managed
5. Services register with Eureka
6. Requests routed using service discovery
7. Monitoring handled by Prometheus
8. Stateless design ensures scalability and fault tolerance

---

## 16. Key Takeaway
> Modern cloud-native applications replace JEE containers with **Docker + Kubernetes**, using **stateless microservices**, **service discovery**, and **orchestrated deployments** to achieve scalability, resilience, and agility.

## Pictorial Representation
-----------------------------
<img width="1114" height="421" alt="Micro Services Clustering and why javaee gone" src="https://github.com/user-attachments/assets/4c69da4c-f188-446a-b720-b0b6d78064e4" />

