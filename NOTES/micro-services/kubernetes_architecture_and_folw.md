# Kubernetes Architecture
-------------------------

## Docker Image
--------------
- A Docker Image is packaged with:
  - Application program
  - Software utilities and tools required to run the application
  - Operating system libraries  
- Using a Docker image, we can run our application in an isolated manner from other applications in the environment.

## Docker Container
------------------
- A Docker image under execution is called a **Docker Container**.

- To manage Docker containers such as:
  - Scaling
  - Deployment
  - Rollouts
  - Availability  
  we use **Kubernetes**.

- In Kubernetes, there are **two main parts**:
  1. Kubernetes Master Node (Control Plane)
  2. Worker Node

---

## 1. Kubernetes Master Node (Control Plane)
-------------------------------------------
- It is the **central engine** of the Kubernetes cluster which takes care of running Pods on the Worker Nodes.
- Ideally, **no Pods are executed on the Master Node**.

- The Kubernetes Master Node plays a **key role** in Kubernetes architecture.
- Responsibilities include:
  - Creating Kubernetes clusters
  - Managing and scaling Worker Nodes
  - Deciding how many Worker Nodes should be added when load increases

---

## 2. Worker Node
----------------
- A **Worker Node** is a machine attached to the Kubernetes Master Node where Pods actually run.
- A Kubernetes cluster can support **up to 5000 Worker Nodes**.

- The Kubernetes Master Node schedules and runs multiple Pods on Worker Nodes.
- To create a Kubernetes cluster, **at least 2 Worker Nodes are required**.

### Processes Running on Worker Node
-----------------------------------
1. **kube-proxy**
   - Manages networking across Pods running in the Kubernetes cluster.

2. **kubelet**
   - Program through which the Master Node communicates with the Worker Node.
   - Responsible for running Pods on the Worker Node.

---

## What is a Pod?
----------------
- A **Pod** is a logical grouping of Docker containers that:
  - Exist together
  - Share common resources

  **OR**

- Multiple Docker containers running together to share:
  - Storage volumes
  - Networking services
  - Other common resources

---

## What is the Size of Your Kubernetes Cluster?
----------------------------------------------
- Assume we have **8 services**, each requiring **1 GB RAM**.
- Worker Nodes are EC2 instances with:
  - 6 GB RAM
  - 2.2 GHz CPU

- Operating System and background services consume **3 GB RAM**.
- Remaining RAM = **3 GB**
- Maximum services per machine = **2 services**

- Required machines:
  - 4 machines to run 8 services (2 services per machine)
  - For High Availability (HA): **4 × 2 = 8 machines**

- Initial cluster capacity = **8 Worker Nodes**

> Note:  
> We can also use high-capacity machines and run all 8 services on a single machine.  
> However, Kubernetes is designed to work efficiently with **smaller-capacity machines** as well.

---

## Flow of Kubernetes
--------------------
- To communicate with the Kubernetes Master Node, we use **kubectl**.
- DevOps engineers provide a **Deployment Manifest**.

### Deployment Manifest Contains:
--------------------------------
1. Docker Image
2. Replicas  
   - Number of containers created from the image
3. Docker Volumes  
   - Volumes shared between containers

### Execution Flow:
------------------
- `kubectl` sends the deployment manifest to the **API Manager**.
- API Manager is a **REST endpoint**.
- API Manager forwards the request to the **Deployment Controller**.
- The **Scheduler**:
  - Communicates with kubelet on Worker Nodes
  - Checks available capacity
  - Schedules Pods on suitable Worker Nodes
- In case of Pod failure:
  - Scheduler creates a new Pod on another Worker Node automatically

---
### Pictorial Representation - Kubernetes Architecture
-------------------------------------------------------
<img width="920" height="350" alt="kubernetes architecture" src="https://github.com/user-attachments/assets/2d4b0519-a3c3-49bc-a765-5ec036c6964e" />

