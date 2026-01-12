# Spring Boot Actuators

## Overview

- When an application is in production, operations engineers need to continuously monitor whether the application health is **up and running**.  
  For this purpose, monitoring tools like **watchdog** are used.

- Traditionally, a developer would have to write a servlet class with an endpoint like `/health`, and the watchdog would periodically send requests to that endpoint.

- Writing such additional monitoring logic manually is **not required** when using the **Spring Boot Actuator** module.

- **Spring Boot applications are production-ready applications.**  
  After development, the developer only needs to **enable the actuator endpoints**.

---

### (OR)

- We can build **development-to-production-grade, readily deployable applications** using the Actuator.

- An application, after completing development, is usually **not ready** to be moved directly to a production environment because production management requires **additional endpoints** to monitor and manage the application.

- Instead of building all such endpoints manually, we can use the **Actuator module provided by Spring Boot**.

---
## Pictorial Representation
<img width="1715" height="563" alt="Screenshot 2026-01-11 162816" src="https://github.com/user-attachments/assets/fec61a24-905d-4f3b-88af-4fd78c1f5f89" />

## What is Actuator?

- Using Spring Boot, we can build applications that are **production-grade and readily deployable**.

- **Actuator** is a module that adds a **bunch of endpoints** that help an application to be easily **monitored and managed** in production environments.

---

## Default Actuator Endpoints

The Actuator module provides many endpoints by default that can be used directly for monitoring and management:

1. `/info` – Provides application information such as application name, version, author, licensing
2. `/health` – Used to check the **liveness probe** of the application
3. `/env` – Displays all environment variables
4. `/beans` – Shows all bean definitions in the IoC container
5. `/configprops` – Displays `@ConfigurationProperties` used in the IoC container
6. `/threaddump` – Displays the current JVM thread dump
7. `/metrics` – Shows memory and CPU usage metrics
8. `/loggers` – Displays loggers and their log levels
9. `/logfile` – Shows application logs
10. `/shutdown` – Shuts down the application remotely
11. `/sessions` – Displays HTTP sessions in a web application
12. `/conditions` – Shows auto-configuration conditions

---

## Maven Dependency

By adding the `spring-boot-starter-actuator` dependency to the project, actuator endpoints become available.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.4.1</version>
</dependency>
````

---

## Endpoint Access Rules

* All actuator endpoints are accessible using the base path:

```
/actuator/{endpointName}
```

* By default:

  * Only `/health` and `/info` are **enabled**
  * All other endpoints are **disabled**

---

## Security Considerations

* By default, actuator endpoints are **not fully enabled** due to security reasons.

* Actuator endpoints are exposed in two ways:

  1. **JMX endpoints**
  2. **HTTP endpoints**

* To enable or disable actuator endpoints, we must configure properties in:

  * `application.properties` or
  * `application.yml`

## Disabling and Enabling Actuator Endpoints

### Disable All Actuator Endpoints

- We can disable **all actuator endpoints** using the following property in Spring Boot.

**application.yml**
```yaml
management:
  endpoints:
    enabled-by-default: false
````

---

### Enable Only One Endpoint

* To enable **only a single actuator endpoint**, disable all endpoints by default and explicitly enable the required one.

**application.yml**

```yaml
management:
  endpoints:
    enabled-by-default: false
  endpoint:
    health:
      enabled: true
```

---

### Enable Multiple Endpoints

* We can enable **multiple actuator endpoints** using the `include` property.

**application.yml**

```yaml
management:
  endpoints:
    web:
      exposure:
        include:
          - health
          - info
          - shutdown
          - beans
          - env
          - configprops
          - logfiles
          - loggers
          - threaddump
```

---

### Include and Exclude Actuator Endpoints

* We can **include and exclude** specific actuator endpoints in a Spring Boot application.

**application.yml**

```yaml
management:
  endpoints:
    web:
      exposure:
        include:        # Endpoints to be included in the application
          - health
          - shutdown
          - beans
          - env
          - configprops
          - logfiles
          - loggers
          - threaddump
        exclude:        # Endpoints to be removed from the application
          - info
          - env
  endpoint:
    shutdown:
      enabled: true
```

---

### Using `application.properties`

* The same configuration can also be done using `application.properties`.

```properties
management.endpoints.web.exposure.exclude=*
management.endpoints.web.exposure.include=health,info,metrics,shutdown
```

* In the above configuration:

  * Only **4 endpoints** are included in the project.
  * The **shutdown** endpoint is **not exposed by default**.
  * Even though the shutdown endpoint exists in the project, it must be **explicitly enabled** to be accessible.


# Full Example

## PracelController.java
```java
package com.ba.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class PracelController {

    @GetMapping("/parcel")
    public String showPracel() {
        return "parcels";
    }
}
````

---

## SpringBootActuatorsApplication.java

```java
package com.ba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootActuatorsApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootActuatorsApplication.class, args);
    }
}
```

---

## parcels.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <title>Parcel</title>
</head>
<body>
    <h2>Parcels</h2>
    <p>Parcels are curior with high technology way.</p>
</body>
</html>
```

---

## application.yml

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp

management:
  endpoints:
#   enabled-by-default: false
    web:
      exposure:
        include:        # Which endpoints should be included in the project
          - health
          - shutdown
          - beans
          - configprops
          - logfiles
          - loggers
          - threaddump
        exclude:        # These endpoints are removed from the project
          - info
          - env
  endpoint:
    shutdown:
      enabled: true
```

---

## Output

* We can access actuator endpoints using the following URL:

```
http://localhost:8080/actuator
```


