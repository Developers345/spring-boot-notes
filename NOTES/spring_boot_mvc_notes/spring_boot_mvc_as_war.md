# How to Deploy a Spring MVC Boot Application as WAR

## Overview

- In general, a **Spring Boot application** runs as a **JAR** file using:

  ```java
  SpringApplication.run(Config.class, args);
````

When this method is executed, the Spring Boot ecosystem starts with an **embedded servlet container** (like Tomcat).

* When deploying a Spring Boot application to a **WildFly application server**, things are different.
  WildFly is a **non-embedded (external) servlet container**, so the application **must be deployed as a WAR** file.

* When a Spring Boot application is packaged as a WAR and deployed to an external servlet container:

  * The servlet container treats it as a **normal WAR deployment**
  * It attempts to register **web application components** (Servlets, Filters, Listeners, etc.)

* However, in a Spring Boot WAR:

  * There is **no `web.xml`**
  * Application bootstrapping normally happens through:

    ```java
    SpringApplication.run(Config.class, args);
    ```

* Therefore, we need a way to trigger the **same bootstrapping logic** when the WAR is deployed to an external servlet container.

---

## What Should We Do to Deploy a Spring Boot Application as WAR?

To support WAR deployment, the main application class must:

1. **Extend `SpringBootServletInitializer`**
2. **Override the `configure()` method**

### Example

```java
@SpringBootApplication
class Application extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }
}
```

---

## How This Works Internally

### Internal Code of `SpringBootServletInitializer`

```java
abstract class SpringBootServletInitializer
        implements WebApplicationInitializer {

    public void onStartup(ServletContext context) {

        SpringApplicationBuilder builder =
                new SpringApplicationBuilder();

        builder = configure(builder);

        SpringApplication springApplication =
                builder.build();

        springApplication.run(args);
    }

    protected abstract SpringApplicationBuilder
            configure(SpringApplicationBuilder builder);
}
```

---

## Key Points to Remember

* `SpringBootServletInitializer` acts as a **replacement for `web.xml`**
* The servlet container calls `onStartup()` during WAR deployment
* `configure()` tells Spring Boot **which application class to bootstrap**
* This ensures the same behavior as `SpringApplication.run()` in a JAR-based deployment

---

✅ With this setup, your **Spring Boot MVC application can run both as**:

* A **standalone JAR**
* A **WAR deployed to external servers like WildFly**

```
````

# Example Program for Spring Boot Application Deployed as WAR

## Sample Program for Spring Boot MVC

---

## Bike.java
```java
package com.boot.mvc.entities;

import jakarta.persistence.*;

@Entity
@Table(name = "bike")
public class Bike {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "bike_no")
    protected int bikeNo;

    @Column(name = "bike_name")
    protected String bikeName;

    @Column(name = "bike_color")
    protected String bikeColor;

    @Column(name = "bike_model")
    protected String bikeModel;

    public Bike() {
    }

    public int getBikeNo() {
        return bikeNo;
    }

    public void setBikeNo(int bikeNo) {
        this.bikeNo = bikeNo;
    }

    public String getBikeName() {
        return bikeName;
    }

    public void setBikeName(String bikeName) {
        this.bikeName = bikeName;
    }

    public String getBikeColor() {
        return bikeColor;
    }

    public void setBikeColor(String bikeColor) {
        this.bikeColor = bikeColor;
    }

    public String getBikeModel() {
        return bikeModel;
    }

    public void setBikeModel(String bikeModel) {
        this.bikeModel = bikeModel;
    }

    @Override
    public String toString() {
        return "Bike{" +
                "bikeNo=" + bikeNo +
                ", bikeName='" + bikeName + '\'' +
                ", bikeColor='" + bikeColor + '\'' +
                ", bikeModel='" + bikeModel + '\'' +
                '}';
    }
}
````

---

## ManagedBikeService.java

```java
package com.boot.mvc.service;

import com.boot.mvc.entities.Bike;
import com.boot.mvc.repositories.BikeRepository;
import jakarta.transaction.Transactional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class ManagedBikeService {

    @Autowired
    private BikeRepository bikeRepository;

    @Transactional
    public List<Bike> getBikes() {
        return bikeRepository.findAll();
    }
}
```

---

## BikeRepository.java

```java
package com.boot.mvc.repositories;

import com.boot.mvc.entities.Bike;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface BikeRepository extends JpaRepository<Bike, Integer> {

    @Override
    List<Bike> findAll();
}
```

---

## BikeController.java

```java
package com.boot.mvc.controller;

import com.boot.mvc.entities.Bike;
import com.boot.mvc.service.ManagedBikeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.List;

@Controller
public class BikeController {

    @Autowired
    private ManagedBikeService managedBikeService;

    @GetMapping("/bike")
    public String bikeList(ModelMap modelMap) {
        List<Bike> bikesList = managedBikeService.getBikes();
        modelMap.addAttribute("bikes", bikesList);
        return "bikes";
    }
}
```

---

## application.yml

```yaml
spring:
  application:
    name: rapido-boot
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/rapido
    username: root
    password: root
  jpa:
    show_sql: true
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp

server:
  servlet:
    context-path: /rapido-boot
```

---

## bikes.jsp

```jsp
<%@ page language="java" %>
<%@ taglib uri="jakarta.tags.core" prefix="c" %>

<html>
<body>
<h2>Bike List</h2>
<table border="1">
    <tr>
        <th>Name</th>
        <th>Model</th>
        <th>Color</th>
    </tr>
    <c:forEach items="${bikes}" var="bike">
        <tr>
            <td>${bike.bikeName}</td>
            <td>${bike.bikeModel}</td>
            <td>${bike.bikeColor}</td>
        </tr>
    </c:forEach>
</table>
</body>
</html>
```

---

## RapidoBootApplication.java

```java
package com.boot.mvc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.domain.EntityScan;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.context.ApplicationContext;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@SpringBootApplication
@EnableJpaRepositories(basePackages = { "com.boot.mvc.repositories" })
@EntityScan(basePackages = { "com.boot.mvc.entities" })
public class RapidoBootApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        ApplicationContext context =
                SpringApplication.run(RapidoBootApplication.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(RapidoBootApplication.class);
    }
}
```

---

## pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.0</version>
        <relativePath/>
    </parent>

    <groupId>boot-mvc</groupId>
    <artifactId>rapido-boot</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>

    <name>rapido-boot</name>
    <description>Demo project for Spring Boot MVC WAR deployment</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <mysql.connector.j.version>9.2.0</mysql.connector.j.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <version>${mysql.connector.j.version}</version>
        </dependency>

        <!-- JSP Support -->
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
        </dependency>

        <dependency>
            <groupId>jakarta.servlet.jsp.jstl</groupId>
            <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
        </dependency>

        <dependency>
            <groupId>org.glassfish.web</groupId>
            <artifactId>jakarta.servlet.jsp.jstl</artifactId>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

---

## Output URL

```
http://localhost:8080/rapido-boot/bike
```
## Pictorial Representation 
<img width="1089" height="362" alt="Screenshot 2026-01-10 175717" src="https://github.com/user-attachments/assets/9a72465e-5474-4da5-b711-ee36ef93093b" />

