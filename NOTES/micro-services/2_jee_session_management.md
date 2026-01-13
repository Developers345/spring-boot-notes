# Pictorial Representation - Session Management in JEE Container
<img width="521" height="361" alt="JEE Server Session Management" src="https://github.com/user-attachments/assets/a1fedbd0-ccdc-46c1-b9f2-1b26eed9ff0d" />

# Session Management in JEE Container
-----------------------------------

## Request Flow Without Sticky Sessions

- User sends a request to the application URL.
- The **Load Balancer** receives the request and forwards it to **Managed Server 2** in `cluster1`.
- **Managed Server 2**:
  - Creates a session for the user.
  - Generates and returns a **JSESSIONID** to the client.
- When the user sends the next request:
  - The load balancer may forward it to **another managed server (e.g., Managed Server 1)**.
  - Since the session object does not exist on that server, the user is redirected to the **login page**.

➡️ This happens because session data is stored **locally** in the managed server where it was created.

---

## Sticky Sessions (Session Affinity)

- To overcome the above problem, configure the load balancer with:
```

sticky-session = true

```
- With sticky sessions enabled:
- All requests from the same user (same JSESSIONID) are always routed to the **same managed server**.
- Benefit:
- Session consistency is maintained.
- Problem:
- **High Availability is lost**.
- If that managed server goes down, the user session is lost.

---

## Session Replication for High Availability

- To achieve **High Availability**, configure **Session Replication** in the cluster.
- With session replication:
- The user session created on one managed server is **replicated to all other managed servers** in the cluster.
- Benefit:
- If one server fails, another server can continue processing the request using the replicated session.
- New Problem Introduced:
- **Scalability issue**
- Example:
  - 1000 users → 1000 session objects
  - Adding one more node means replicating all 1000 session objects
  - This leads to higher memory usage and network overhead

---

## Responsibilities of JEE Container in Managing an Enterprise Application
-------------------------------------------------------------------------

### 1. Cluster Management
- Create managed servers
- Remove managed servers
- Monitor the health of servers in the cluster

### 2. Start / Stop of Cluster
- Start the entire cluster or individual managed servers
- Graceful shutdown and restart handling

### 3. Application Deployment
- Deploy applications across all managed servers in the cluster
- Ensure consistent application versions

### 4. Load Balancing
- Distribute incoming requests across managed servers
- Improve performance and availability

### 5. Service Discovery and Routing
- Discover available services in the cluster
- Route traffic to the appropriate application instance

### 6. Session Management
- Session replication
- Fault tolerance
- High availability

---

## Key Definitions
----------------

### High Availability (HA)
- Even if one server goes down, **another server is available** to handle user requests.

### Fault Tolerance
- If a server fails **while processing a request**, the request is processed by another server using **replicated session data**.
- End users experience **no interruption**.

---

## Summary
--------

- Sticky sessions solve session consistency but reduce availability.
- Session replication improves availability but impacts scalability.
- A well-designed JEE cluster balances **performance, availability, and scalability**.

