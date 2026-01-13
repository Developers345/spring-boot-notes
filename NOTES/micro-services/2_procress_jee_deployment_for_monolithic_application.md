# Pictorial Representation -JEE Container Perspective – Cluster Deployment
<img width="1093" height="409" alt="JEE Server Cluster Deployment" src="https://github.com/user-attachments/assets/6cb53ba2-1ab0-4134-97d4-e5a6732e7ab8" />

# JEE Container Perspective – Cluster Deployment

## Domain Concept
- In a JEE server, we create **domains** (WebLogic) or **profiles** (WebSphere).  
  These hold the application configurations with which an application is deployed on the server.

**OR**

- A **domain** can be viewed as a logical grouping of application configurations with which we want to run our application.  
  These configurations are **not available** to applications outside the domain.

---

## Servers Inside a Domain
Within a domain, we create servers. There are **two types of servers**:

---

## 1. Admin Server

### Duties / Responsibilities of Admin Server
a. **Domain configuration management**
   - Configure DataSources, security configurations, etc.
   - Configuration access methods:
     - Administrative Console  
     - JMX (programmatic approach to connect JEE server)  
     - WLST scripts (programmatic approach specific to WebLogic)

b. **Creating / deleting managed servers** across clusters

c. **Cluster configuration**
   - Creating clusters  
   - Deleting clusters

d. **Application deployment**
   - Applications are deployed through the Admin Server

---

### Important Note (VIMP)
- Once configurations are completed, the **Admin Server is stopped** so no one can modify configurations.
- Applications continue to run on **Managed Servers**.
- In production environments, deployers usually **stop the Admin Server**.

**Example:**
- A DataSource is configured at the domain level via the Admin Server.
- The Admin Server asks which Managed Server(s) to target.
- The DataSource is configured for a specific Managed Server.
- Only that Managed Server can access the DataSource; others cannot.

- Applications always run on **Managed Servers**.
- Hence, deployments are **highly secure**.

---

## 2. Managed Server

a. Applications are deployed and accessed on Managed Servers.  
   - One application per Managed Server **OR**  
   - Multiple applications per Managed Server

---

## Cluster & Node Architecture

- On a machine, we install a JEE server (WebLogic / JBoss).
- Inside the JEE server, we create a **domain**.
- Inside the domain, we create an **Admin Server**.
- The machine hosting the Admin Server acts as the **Master Node**.

- Generally, the Admin Server node does **not** host Managed Servers.
- However, due to cost considerations, companies often install **one Managed Server** on the Admin Server (Master Node).

- Remaining machines/nodes:
  - Install JEE container (WebLogic / WebSphere)
  - Contain **only Managed Servers**, no Admin Server

- From the Admin Server (Master Node):
  - Configure clusters
  - Configure Managed Servers
  - Based on this configuration, the JEE container creates clusters and adds Managed Servers to them

- Admin Server is available only at **static/configuration time**, not at runtime.

---

## Scaling & Cluster Management

- Adding a new node to the cluster is easy:
  - Package the domain configuration from an existing Managed Server node
  - Unpack it on a new machine
  - Add the new node to the cluster using Admin Server configuration

- Clusters are created by the Admin Server.
- Each cluster maintains information about its Managed Servers:
  - IP addresses
  - Port numbers
  - Other metadata

---

## Application Deployment in Cluster

- Application (e.g., `rapido.ear`) is given to the Admin Server.
- The Admin Server (Master Node) automatically deploys the application to **all Managed Servers** in the cluster.

---

## Node Manager & Load Balancer

- Each Managed Server has a **Node Manager**.
- The cluster uses Node Manager to:
  - Monitor Managed Server health

- Node Manager periodically checks Managed Servers and reports:
  - Which servers are **UP**
  - Which servers are **DOWN**

- Load Balancer is directly attached to the **cluster**.
- Cluster notifies the Load Balancer about server health using Node Manager information.

- User requests flow:
  1. User sends request
  2. Load Balancer receives the request
  3. Load Balancer consults cluster status
  4. Traffic is routed to an available Managed Server
     - Using algorithms like **Round Robin**
