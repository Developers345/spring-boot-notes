# Spring Boot JPA

## Overview
- Unlike Spring Data JPA, **Spring Boot Data JPA auto-configuration** takes care of the following framework classes:
  1. `DataSource`
  2. `EntityManagerFactory`
  3. `TransactionManager`
  4. `JpaTemplate`  
     - *(Available in Spring 5.x; completely removed from Spring 6.x)*

---

## Example Program for Spring Boot JPA

---

## Bike.java (Entity Class)
```java
package com.boot.datajpa.entities;

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
package com.boot.datajpa.service;

import com.boot.datajpa.entities.Bike;
import com.boot.datajpa.repositories.BikeRepository;
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

    @Transactional
    public List<Bike> getBikesByName(String bikeName) {
        return bikeRepository.findByBikeName(bikeName);
    }

    public Bike getBikeNameAndBikeModel(String bikeName, String bikeModel) {
        return bikeRepository.getByBikeNameModel(bikeName, bikeModel);
    }

    public Bike getBikeNameAndBikeModelAndColor(
            String bikeName,
            String bikeModel,
            String bikeColor) {

        return bikeRepository.getByBikeNameModelColor(
                bikeName, bikeModel, bikeColor);
    }
}
```

---

## BikeRepository.java

```java
package com.boot.datajpa.repositories;

import com.boot.datajpa.entities.Bike;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface BikeRepository extends JpaRepository<Bike, Integer> {

    @Override
    List<Bike> findAll();

    List<Bike> findByBikeName(String bikeName);

    @Query("from Bike b where b.bikeName=?1 and b.bikeModel=?2")
    Bike getByBikeNameModel(String bikeName, String bikeModel);

    @Query("from Bike b where b.bikeName=:bikeName and b.bikeModel=:bikeModel and b.bikeColor=:bcolor")
    Bike getByBikeNameModelColor(
            String bikeName,
            String bikeModel,
            @Param("bcolor") String bikeColor);
}
```

---

## application.yml

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/rapido
    username: root
    password: root
  jpa:
    show_sql: true
    generate-ddl: true
```

---

## SpringBootDataJpaApplication.java

```java
package com.boot.datajpa;

import com.boot.datajpa.entities.Bike;
import com.boot.datajpa.service.ManagedBikeService;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.domain.EntityScan;
import org.springframework.context.ApplicationContext;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

import java.util.List;

@SpringBootApplication
@EnableJpaRepositories(basePackages = {"com.boot.datajpa.repositories"})
@EntityScan(basePackages = {"com.boot.datajpa.entities"}) // mapping-info -> JPA
public class SpringBootDataJpaApplication {

    public static void main(String[] args) {

        ApplicationContext context =
                SpringApplication.run(SpringBootDataJpaApplication.class, args);

        ManagedBikeService managedBikeService =
                context.getBean("managedBikeService", ManagedBikeService.class);

        List<Bike> bikesByName =
                managedBikeService.getBikesByName("Honda");

        bikesByName.forEach(System.out::println);

        Bike bikeNameModel =
                managedBikeService.getBikeNameAndBikeModel("Honda", "B1");
        System.out.println(bikeNameModel);

        Bike bikeNameModelColor =
                managedBikeService.getBikeNameAndBikeModelAndColor(
                        "Honda", "B1", "Red");
        System.out.println(bikeNameModelColor);
    }
}
```

---

## Output

```text
Hibernate: select b1_0.bike_no,b1_0.bike_color,b1_0.bike_model,b1_0.bike_name 
from bike b1_0 where b1_0.bike_name=?
Bike{bikeNo=1, bikeName='Honda', bikeColor='Red', bikeModel='B1'}

Hibernate: select b1_0.bike_no,b1_0.bike_color,b1_0.bike_model,b1_0.bike_name 
from bike b1_0 where b1_0.bike_name=? and b1_0.bike_model=?
Bike{bikeNo=1, bikeName='Honda', bikeColor='Red', bikeModel='B1'}

Hibernate: select b1_0.bike_no,b1_0.bike_color,b1_0.bike_model,b1_0.bike_name 
from bike b1_0 where b1_0.bike_name=? and b1_0.bike_model=? and b1_0.bike_color=?
Bike{bikeNo=1, bikeName='Honda', bikeColor='Red', bikeModel='B1'}
```
