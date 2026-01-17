# Practical Implementation
---------------------------

→ We can build a **microservice-based architecture** using many technologies, but **Spring Boot** provides a very easy and efficient way to build microservice applications.

→ To create a **Eureka Server** using other technologies, we must download the Eureka JAR and configure it manually, which is time-consuming.  
Spring Boot simplifies this by providing a starter dependency:

```

spring-cloud-starter-netflix-eureka-server

````

This makes Eureka Server development **easy and fast**.

---

## Creating the Eureka Server through Spring Boot  
### (rapido-eureka-server)
-----------------------------------------------

### 1. Project Name
- `rapido-eureka-server`

### 2. Add the Dependencies
- Eureka Server

---

### Eureka Server Behavior

→ By default, the Eureka Server tries to identify itself as a client and registers with itself.

→ But in our case, **we only want clients to register with the Eureka Server**, not the server itself.

So we disable:
- `register-with-eureka`
- `fetch-registry`

---

### `application.yml`
```yaml
server:
  port: 8761

eureka:
  client:
    eureka-server-port: 8761
    register-with-eureka: false   # Do not register Eureka Server as a client
    fetch-registry: false         # Do not fetch client registry
````

→ Using the above configuration, the Eureka Server starts **without client discovery**.

→ **Default Eureka Server port:** `8761`

---

### Enable Eureka Server

In the Spring Boot application, enable:

```java
@EnableEurekaServer
```

---

### Notes on Port Configuration

```yaml
server:
  port: 8761

eureka:
  client:
    eureka-server-port: 8761
```

→ If Tomcat runs on port **8761**, there is **no need** to explicitly configure the Eureka Server port.

→ If Tomcat runs on a **different port**, configure the Eureka Server port to match the Tomcat port.

---

### Access Eureka Server

```
http://localhost:8761/
```

---

## (rapido-user-service)

---

### Enabling Spring Data JPA

#### `application.yml`

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/rapido_userdb
    username: root
    password: root

  jpa:
    show-sql: true
    generate-ddl: true
```

---

### Change Embedded Tomcat Port

#### `application.yml`

```yaml
server:
  port: 8081
```

---

## Registering Microservices with Eureka Server

---

### 1. Add Dependency

* **Eureka Discovery Client**

### 2. `application.yml`

```yaml
spring:
  application:
    name: user-service

eureka:
  client:
    register-with-eureka: true   # Register with Eureka Server
    service-url:
      defaultZone: ${EUREKA_URI:http://localhost:8761/eureka}
```

→ The client automatically registers with the Eureka Server.

---

### Enabling Discovery Client

```java
//@EnableDiscoveryClient
```

→ Mandatory in **Spring Boot 1.x & 2.x**
→ **Optional from Spring Boot 3.x & Spring Cloud 2024.x**

---

### Microservice Identification

```yaml
spring:
  application:
    name: user-service
```

→ The **service name** is how the microservice is identified in the Eureka Server.

---

## rapido-mobile-platform

### (Client Application – Consumes Data)

---

### 1. Add Dependency

* **Eureka Discovery Client**

### 2. REST Call

```java
restTemplate.getForEntity(
  "http://USER-SERVICE/api/customers",
  Customer[].class
);
```

### 3. `application.yml`

```yaml
spring:
  application:
    name: rapido-mobile-platform

eureka:
  client:
    register-with-eureka: false
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8761/eureka
```

### 4. Enable Eureka Client

```java
@EnableEurekaClient
```

→ Ensures REST client always goes to Eureka Server for service discovery.

---

## Spring Boot 3.4.x + Microservices

---

### ❗ `@EnableEurekaClient` is NO LONGER REQUIRED

---

## Why `@EnableEurekaClient` is NOT Needed

### Old Behavior (Spring Boot 1.x / 2.x)

```java
@EnableEurekaClient
@SpringBootApplication
public class OrderServiceApplication { }
```

→ Without this annotation, the service would **not register**.

---

### New Behavior (Spring Boot 3.x + Spring Cloud 2024.x)

✔ Auto-configuration enables Eureka automatically
✔ Dependency + configuration is enough

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

---

## Equivalent Annotation?

✔ **No replacement annotation**
✔ `@SpringBootApplication` is sufficient

---

## Configuration That Matters

```yaml
spring:
  application:
    name: order-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
```

---

## Annotation Status Table

| Annotation             | Status      |
| ---------------------- | ----------- |
| @EnableEurekaClient    | ❌ Removed   |
| @EnableDiscoveryClient | ⚠️ Optional |
| Auto-configuration     | ✅ Default   |

---

## Final Rule (Important)

> **In Spring Boot 3.x microservices, annotations are replaced by auto-configuration.**

---

## Client Application Code Example

```java
@SpringBootApplication
public class RapidoMobilePlatformApplication {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {

        ConfigurableApplicationContext context =
            SpringApplication.run(RapidoMobilePlatformApplication.class, args);

        UserService userService =
            context.getBean("userService", UserService.class);

        List<Customer> customers = userService.getCustomers();
        customers.forEach(System.out::println);
    }
}
```

---

### Internal Flow

→ REST client goes to Eureka Server using:

```
http://localhost:8761/eureka
```

→ Eureka Server replaces:

```
USER-SERVICE
```

with the actual service URL:

```
http://USER-SERVICE/api/customers
```

→ Client-side load balancer routes the request using **RestTemplate**.

---
### Pictorial Representation - Eureka server flow - 1
<img width="1911" height="732" alt="Screenshot 2026-01-16 123250" src="https://github.com/user-attachments/assets/2cbbd3c3-6f8b-4a35-9ca4-6cb196a282d8" />

### Pictorial Representation - Eureka server flow - 2
<img width="1422" height="659" alt="Screenshot 2026-01-16 123313" src="https://github.com/user-attachments/assets/3e4d5508-3dde-43cd-9d31-e8b103af72c8" />
