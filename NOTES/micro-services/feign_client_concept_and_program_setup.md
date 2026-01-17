# Feign Client with Eureka – Detailed Understanding
-----------------------------------------------

The diagram explains how **Spring Cloud OpenFeign** works with **Eureka Service Discovery**
to enable **declarative REST communication** between microservices.

---

## Components in the Diagram
----------------------------

### 1. rapido-mobile-platform (Client Application)
- Acts as a **consumer microservice**
- Uses **Feign Client** to call another microservice
- Does NOT hardcode service URLs

---

### 2. rapido-user-service (Provider Application)
- Acts as a **producer microservice**
- Registers itself with **Eureka Server**
- Exposes REST endpoints (e.g., `/customers`)

---

### 3. Eureka Server
- Acts as a **service registry**
- Maintains:
  - Service names
  - Instance metadata (host, port)
- Enables **service discovery**
- Does NOT perform load balancing

---

## Dependencies (pom.xml)
------------------------

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
````

### Transitive Dependencies

* `spring-cloud-loadbalancer`
* HTTP client (Apache / OkHttp / default)
* Eureka discovery support (if present)

→ **Feign automatically integrates with LoadBalancer**

---

## Enabling Feign in Spring Boot

---

```java
@SpringBootApplication
@EnableFeignClients
public class RapidoMobilePlatformApplication {
}
```

### What `@EnableFeignClients` Does

* Scans interfaces annotated with `@FeignClient`
* Creates **runtime proxy implementations**
* Registers them as Spring beans
* Enables dependency injection

---

## Feign Client Interface

---

```java
@FeignClient(name = "USER-SERVICE")
public interface CustomerService {

    @GetMapping(
      value = "/customers",
      produces = "application/json"
    )
    List<Customer> customers();
}
```

### Important Points

* `name = "USER-SERVICE"`

  * This is the **Eureka service ID**
* No base URL required
* REST mapping looks like a controller
* Feign hides:

  * HTTP calls
  * Serialization / deserialization
  * Error handling

---

## How Service Discovery Works

---

1. `rapido-user-service` starts
2. Registers itself with Eureka
3. Eureka stores instance details:

   * Host
   * Port
   * Service name

```yaml
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
```

---

## Client-Side Invocation Flow

---

```text
Feign Client
   ↓
Spring Cloud LoadBalancer
   ↓
Eureka Server
   ↓
Resolved Service Instance
   ↓
HTTP Call to Service
```

---

## Runtime Execution Flow

---

```java
ConfigurableApplicationContext context =
    SpringApplication.run(RapidoMobilePlatformApplication.class, args);

CustomerService cs =
    context.getBean("customerService", CustomerService.class);

List<Customer> customers = cs.customers();
```

### Internally:

* Spring injects **Feign proxy**
* Proxy intercepts method call
* LoadBalancer selects instance
* HTTP request is executed
* Response mapped to `List<Customer>`

---

## Load Balancing Behavior

---

* Feign uses **Spring Cloud LoadBalancer**
* Default strategy:

  * Round-robin
* Works across:

  * Multiple instances
  * Dynamic scaling

---

## Why Feign is Better than RestTemplate

---

| Feature           | RestTemplate | Feign |
| ----------------- | ------------ | ----- |
| Boilerplate code  | High         | Low   |
| Service discovery | Manual       | Auto  |
| Load balancing    | Manual       | Auto  |
| Declarative style | ❌            | ✅     |
| Readability       | Medium       | High  |

---

## Configuration in User Service

---

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka
    register-with-eureka: true
    fetch-registry: true
```

→ Allows the service to:

* Register with Eureka
* Be discoverable by Feign clients

---

## Important Notes (Spring Boot 3.x)

---

* `@EnableFeignClients` **IS required**
* `@EnableEurekaClient` **NOT required**
* Discovery is enabled via:

  * Dependency
  * Auto-configuration

---

## Common Mistakes

---

* ❌ Service name mismatch in `@FeignClient`
* ❌ Forgetting `@EnableFeignClients`
* ❌ Missing Eureka client dependency
* ❌ Not exposing correct REST endpoint path

---

## Key Takeaways

---

* Feign provides **declarative REST clients**
* Eureka provides **service discovery**
* LoadBalancer provides **client-side load balancing**
* Together they enable:

  * Loose coupling
  * Scalability
  * Clean microservice communication

---
# Feign Client
-------------

→ **Feign Client** is a third-party library that **Spring Boot integrates and provides**
as **Spring Cloud OpenFeign**.

→ Feign Client supports **declarative client programming** for calling remote
microservices.

---

## Key Characteristics
-----------------------

→ Feign Client follows a **declarative approach**, meaning:
- No manual HTTP client code
- No boilerplate REST call logic
- No direct usage of `RestTemplate` or `WebClient` for service-to-service calls

---

## IoC Container Behavior
-------------------------

→ During **IoC container creation**:
- Spring scans for interfaces annotated with `@FeignClient`
- For each such interface:
  - Spring creates a **runtime proxy implementation**
  - Registers it as a Spring bean

→ These proxy objects handle:
- Service discovery
- Load balancing
- HTTP communication
- Serialization and deserialization

---

## Replacing RestTemplate
-------------------------

→ With Feign Client:
- We **do not need** to invoke microservices using `RestTemplate`
- Method calls on Feign interfaces internally perform REST calls

Example concept:
```text
customerService.getCustomers();
````

→ Internally translated into an HTTP request.

---

## Declarative Programming Model

---

→ Feign Client allows us to:

* Define an **interface**
* Describe remote service endpoints as methods
* Decorate methods with Spring MVC annotations

→ We only describe **what to call**, not **how to call**.

---

## Example Concept

---

```java
@FeignClient(name = "user-service")
public interface UserService {

    @GetMapping("/users")
    List<User> getUsers();
}
```

→ No implementation class required
→ Feign generates implementation at runtime

---

## Customization & Configuration

---

Feign Client is highly configurable and customizable:

### 1. Encoders

* Convert Java objects into HTTP request bodies
* Example: JSON request serialization

### 2. Decoders

* Convert HTTP responses into Java objects
* Example: JSON → POJO mapping

### 3. Loggers

* Log request and response details
* Useful for debugging and monitoring

### 4. Load Balancing

* Integrates with:

  * Spring Cloud LoadBalancer
  * Eureka service discovery
* Distributes requests across service instances

---

## Advantages of Feign Client

---

* Declarative and clean code
* Less boilerplate
* Easy integration with Eureka
* Built-in load balancing
* Improves readability and maintainability

---

## Summary

---

* Feign Client is a **declarative REST client**
* Eliminates manual REST invocation code
* Uses interfaces + annotations
* Runtime proxy creation by IoC container
* Highly configurable and extensible


## Pictorial Representation - 1
<img width="1348" height="491" alt="fiegn setup completed" src="https://github.com/user-attachments/assets/dbb4cbfd-7030-49c9-85ca-d44cfe2af057" />

## Pictorial Representation - 2
<img width="1886" height="718" alt="Screenshot 2026-01-16 185916" src="https://github.com/user-attachments/assets/220cc741-f3be-494b-b5a0-fc64bae572da" />
