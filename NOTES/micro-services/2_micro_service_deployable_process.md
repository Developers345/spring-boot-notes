# Execution Flow of the Micro-services in Kubernetes

---

- **Developer** develops the micro-services applications and packages each micro-service as a **Docker image**.  
  These Docker images are then provided to the **MicroService-Deployer**.

- **MicroService-Deployer** configures the **Kubernetes master node** with:
  - Docker images of all micro-services  
  - Maximum number of worker nodes to be created  

- **Kubernetes master node** creates the worker nodes based on the configuration and deploys **pods** on the worker nodes.  
  For **high availability**, Kubernetes ensures that at least **two pods run on two different worker nodes**.

- Here, we **cannot use a traditional Load Balancer** because:
  - A load balancer routes traffic only to nodes in the cluster
  - In micro-services architecture, services are **scattered across multiple nodes**
  - Hence, the load balancer cannot identify which service is running on which node

- To **discover services**, we use a **Service Discovery Registry** called **Eureka Server**.  
  The developer creates the Eureka Server and runs it in Kubernetes as a separate pod, called the **Eureka Pod**.

- The developer writes code in each micro-service to **register itself** with the Eureka Server by sending:
  - IP address  
  - Port number  
  - Service name  

- All pods running on worker nodes **periodically send heartbeat signals** to the Eureka Server to indicate that the micro-services are alive.  
  If any failure occurs in production, the Eureka Server is aware of it.

- The **job of the Eureka Server** is:
  - When a client sends a request, Eureka discovers how many instances (pods) of the required micro-service are running in the cluster
  - It returns this information to the client

- On the **client side**, a load balancer called **Ribbon** is used.  
  **Ribbon is a client-side load balancer** whose job is:
  - To receive the service instance information from Eureka
  - To route requests based on load-balancing algorithms such as **Round Robin**

- **Zuul** acts as a **client-side API Gateway**, which sends client requests to the appropriate backend services.

- **Hystrix** is a **Circuit Breaker**, used to handle failures gracefully and prevent cascading failures in micro-services.

# Pictorial Representation 
<img width="1351" height="523" alt="Micro Services deployable process in kubernetes" src="https://github.com/user-attachments/assets/d592c98b-4292-4c05-8d15-316b5d7c88b1" />

