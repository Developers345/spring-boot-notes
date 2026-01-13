# Process of Learning Microservices

## Monolithic Application

- What does a JEE container do?
- How are applications managed in the JEE world?
- What challenges do we face when we deploy a single deployment artifact?
- How can we break applications into modules and make them independently deployable solutions that run on multiple machines?
- What are the disadvantages of a microservices architecture and how can we overcome them?

---

## Enterprise Application Overview

- There is an enterprise application that comprises several modules.
- There are different types of clients accessing the application, for example:
  1. Desktop application  
  2. Web application  
  3. Third-party application integration  
  4. Exposed APIs  
  5. Mobile applications  

- The application interacts with a backend database and exchanges messages between modules.
- We can build these types of applications using **monolithic application development / architecture**.

---

## Deployment Model (Monolithic)

- We can deploy the application as a **single deployable artifact**.
- The application contains many modules such as Java modules or Web modules, etc.
- In Java, we typically deploy the application as an **`.ear`** file into an application server such as:
  - WebLogic  
  - JBoss  
  - GlassFish  

---

## Services Provided by a JEE Container

A JEE container provides several services while running the application:

1. Clustering  
2. Load balancing  
3. Security  
4. Transactionality  
5. Logging  
6. Connection pooling  
7. Monitoring  
8. Messaging support  

---

## Services Offered by Modern Frameworks (Without Full JEE Server)

With modern frameworks, some services are provided directly without a full JEE server:

1. Connection pooling  
2. Security  
3. Monitoring  
4. Messaging support  
5. Logging  

---

## Services That Still Require JEE Container Support

Some container-level services still typically require JEE container support:

1. Scalability  
2. Transactionality
   
## Pictorial Representation of monolithic sigle deployable artifact
<img width="1893" height="669" alt="Monolithic single deployable artifact" src="https://github.com/user-attachments/assets/9486105d-6d82-417a-8b7b-199de0d94a97" />

