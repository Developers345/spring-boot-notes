## Enterprise Application Packaging and Deployment

- An enterprise application that consists of several modules is packaged and deployed as a **single deployable artifact**.

### Example Application
-----------------------

**Rapido (Application)**

**Customer**
- User Management  
- Bike  
- Rental  
- MyAccount  
- Payment  
- Support  

**Admin**
- Bike Management  
- Account Verification  
- Refunds  
- Complaints  
- Notifications  
- Reports / Statements  

---

## Application Packaging as EAR
--------------------------------

- The above application is packaged as a **single EAR deployment** and deployed on a **JEE container** as shown below:

```

## rapido.ear

├── customer.war
├── admin.war
└── common.war   (common files used by both customer and admin WARs as a dependency)

```

---

## Deployment Architecture in a JEE Container
----------------------------------------------

- The application is deployed on a **JEE container** as a single deployable artifact.
- For **scalability** and **high availability**, we rely on the **clustering capabilities of the JEE container**.

---

## Clustering Perspective of Non-JEE Containers / Servers
----------------------------------------------------------

### Non-Clustered Deployment
----------------------------

- The application is deployed on a single machine and users access it directly.
- If a large number of requests arrive, the application may fail to handle them because the **machine capacity is insufficient** due to **limited computing resources** (CPU, memory, etc.).

---

### Clustered Deployment
------------------------

- In a cluster environment, the application is scaled across **multiple computers/machines**, where each computer is called a **Node**.
- The application is deployed on **two or more nodes** in the cluster.
- A problem arises when users try to access the application using **multiple domain names** (e.g., `rapido1`, `rapido2`).

- To overcome this problem, we use a **Load Balancer** (or **HTTP Front End**).
  - Users access the application using the **Load Balancer URL**.
  - The Load Balancer forwards requests to cluster nodes using algorithms such as **Round Robin**.

---

## Benefits of Clustered Environment
------------------------------------

### 1. Scalability
------------------
- By adding more nodes to the cluster, the application can handle an increased number of user requests.

### 2. High Availability
-----------------------
- If one node goes down, service disruption does not occur.
- Other nodes in the cluster continue to handle incoming traffic.
  
### Pictorial Representation - diff non-cluster and cluster deployment
<img width="1175" height="706" alt="Differernce non-cluster and cluster" src="https://github.com/user-attachments/assets/c7ecde09-4ae5-4c0e-af58-0424bce9494b" />

### Pictorial Representation - Cluster deployment 
<img width="1466" height="578" alt="General Cluster Deployment" src="https://github.com/user-attachments/assets/40bcc2ef-4758-4f20-9ae7-69981fdad108" />
