# Spring Boot MVC

## Sample program for Spring Boot MVC

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

```yml
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
import org.springframework.context.ApplicationContext;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@SpringBootApplication
@EnableJpaRepositories(basePackages = { "com.boot.mvc.repositories" })
@EntityScan(basePackages = { "com.boot.mvc.entities" })
public class RapidoBootApplication {

    public static void main(String[] args) {
        ApplicationContext context =
                SpringApplication.run(RapidoBootApplication.class, args);
    }
}
```

---

## Output

```
http://localhost:8080/rapido-boot/bike
```
<img width="355" height="352" alt="Screenshot 2026-01-09 181638" src="https://github.com/user-attachments/assets/5ec00db7-6cd7-4c90-bbab-988e5a9e39be" />

## Pictorial Representation 
<img width="1603" height="492" alt="Screenshot 2026-01-08 205939" src="https://github.com/user-attachments/assets/3bce24f6-429f-4718-a963-1e185306b7f5" />

