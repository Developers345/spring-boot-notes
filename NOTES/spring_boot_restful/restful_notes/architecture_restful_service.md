Architecture of RESTful Service
-------------------------------
### Pictorial Representation - Architecture of RESTful Service:
<img width="3217" height="1921" alt="ARCH_RESTFULSERVICE" src="https://github.com/user-attachments/assets/9236bf3f-56ac-441a-b36a-f7f7debbe122" />

### Note:
------

- We can build RESTful service resources using Java.  
  However, we **cannot directly send Java objects** to the client because Java objects must be converted into **bits and bytes** using **serialization**.

- Java serialization is **Java-specific**.  
  If the client application is developed using a different technology (for example, **.NET**), Java serialization is **not supported**.

- To follow the **interoperability principle** in REST, data should be represented using **standard data representation formats**, such as:
  - JSON  
  - XML  
  - YAML  
  - CSV  

---

- To work with RESTful services in Java, Java provides an API called **JAX-RS API**.

- APIs are only **specifications**. Third-party vendors provide implementations for these Java APIs.

- There are two popular JAX-RS implementations:
  1. **Jersey** – Reference implementation provided by Oracle  
  2. **RESTEasy** – Provided by third-party vendor JBoss  

---

- The **Spring Framework** also provides its **own implementation** for building RESTful services.

### Benefits of using Spring implementation:

1. We can leverage **IoC container services** as part of the REST components we are building.
2. Provides **closer integration with Spring Cloud** and **Microservices-based applications**.

### Pictorial Representation - Technical Architecture of RESTful Service:
<img width="3217" height="1921" alt="TECHNICAL_ARCH_RESTFUL" src="https://github.com/user-attachments/assets/92048b31-daa6-441d-918f-6ce50382c507" />

### Pictorial Representation - RESTful Service Example:
<img width="3217" height="1921" alt="BASIC_REST_EXAMPLE" src="https://github.com/user-attachments/assets/cd49254b-9a98-4940-bc7a-acb6d012ca80" />

