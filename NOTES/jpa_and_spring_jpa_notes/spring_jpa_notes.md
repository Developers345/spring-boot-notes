# Spring JPA
-----------

### Problems with Plain JPA (Boilerplate Code)

1. Writing code to instantiate the `EntityManagerFactory`
2. Managing only **one `EntityManagerFactory` per database** using helper classes
3. Creating `EntityManager` and closing it after usage
4. Closing `EntityManagerFactory` at the end of the application
5. Writing explicit code for **transaction management**

👉 To reduce the above boilerplate logic, **Spring JPA** comes into the picture.

---

## Example Program
------------------

## BikeEntity.java
-----------------
```java
package com.rapido.entities;

import jakarta.persistence.*;

@Entity
@Table(name = "bike")
@NamedQuery(
        name = "getBikes",
        query = "from Bike"
)
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

## BikeDAO.java

---

```java
package com.rapido.dao;

import com.rapido.entities.Bike;
import jakarta.persistence.EntityManager;
import jakarta.persistence.PersistenceContext;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
/*
 According to the JPA specification:

 @NamedQuery MUST be defined on an @Entity class
 OR in META-INF/orm.xml

 @NamedQueries({
     @NamedQuery(name = "getBikes", query = "from Bike")
 })
*/
public class BikeDao {

    /*
     JpaTemplate is removed from Spring 6.x onwards
    */
    @PersistenceContext
    private EntityManager entityManager;

    public List<Bike> getBikes() {
        return entityManager
                .createNamedQuery("getBikes", Bike.class)
                .getResultList();
    }
}
```

---

## ManageBikeService.java

---

```java
package com.rapido.service;

import com.rapido.dao.BikeDao;
import com.rapido.entities.Bike;
import jakarta.transaction.Transactional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class ManageBikeService {

    @Autowired
    private BikeDao bikeDao;

    @Transactional
    public List<Bike> getBikes() {
        return bikeDao.getBikes();
    }
}
```

---

## PersistenceJavaConfig.java

---

```java
package com.rapido.config;

import jakarta.persistence.EntityManagerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.*;
import org.springframework.core.env.Environment;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;
import java.util.Properties;

@Configuration
@PropertySource("classpath:db.properties")
@ComponentScan(basePackages = {"com.rapido.dao", "com.rapido.service"})
@EnableTransactionManagement
public class PersistenceJavaConfig {

    @Autowired
    private Environment env;

    @Bean
    public DataSource dataSource() {

        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(env.getProperty("db.driverClassName"));
        dataSource.setUrl(env.getProperty("db.url"));
        dataSource.setUsername(env.getProperty("db.username"));
        dataSource.setPassword(env.getProperty("db.password"));

        return dataSource;
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryBean(
            DataSource dataSource) {

        LocalContainerEntityManagerFactoryBean emfBean =
                new LocalContainerEntityManagerFactoryBean();

        Properties props = new Properties();
        props.put("hibernate.show_sql", "true");

        emfBean.setDataSource(dataSource);
        emfBean.setJpaProperties(props);
        emfBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        emfBean.setPackagesToScan("com.rapido.entities");

        return emfBean;
    }

    @Bean
    public JpaTransactionManager jpaTransactionManager(
            EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
}
```

---

## db.properties

---

```properties
db.driverClassName=com.mysql.cj.jdbc.Driver
db.url=jdbc:mysql://localhost:3306/rapido
db.username=root
db.password=root
```

---

## SpringJpaTest.java

---

```java
package com.rapido.test;

import com.rapido.config.PersistenceJavaConfig;
import com.rapido.entities.Bike;
import com.rapido.service.ManageBikeService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import java.util.List;

public class SpringJpaTest {

    public static void main(String[] args) {

        ApplicationContext context =
                new AnnotationConfigApplicationContext(PersistenceJavaConfig.class);

        ManageBikeService manageBikeService =
                context.getBean("manageBikeService", ManageBikeService.class);

        List<Bike> bikes = manageBikeService.getBikes();
        bikes.forEach(System.out::println);
    }
}
```

---

## Output

---

```text
Hibernate: select b1_0.bike_no,b1_0.bike_color,b1_0.bike_model,b1_0.bike_name from bike b1_0
Bike{bikeNo=1, bikeName='Honda', bikeColor='Red', bikeModel='B1'}
Bike{bikeNo=2, bikeName='Hero', bikeColor='Blue', bikeModel='B2'}
Bike{bikeNo=3, bikeName='Hero Honda', bikeColor='White', bikeModel='B3'}
Bike{bikeNo=4, bikeName='Pluser', bikeColor='Green', bikeModel='B4'}
```
# Internal Flow of Spring JPA
---------------------------

```java
jpaTemplate.persist(bike);
// jpaTemplate is an old one.
// Instead of JpaTemplate, directly use EntityManager with @PersistenceContext
````

### Internal Call Flow

```
jpaTemplate.persist(bike)
   |
   └─ internally calls EntityManager
        |
        ├─ entityManager = entityManagerFactory.createEntityManager();
        ├─ entityManager.persist(bike);
        |
        └─ EntityManagerFactory
             [mapping info, connection pool]
             |
             └─ PersistenceContext.createEntityManagerFactory("rapidoPU")
```

---

## Why EntityManagerFactory Is Required

* We **cannot create `EntityManager` directly**.
* So we must go through **EntityManagerFactory**.
* Creating `EntityManagerFactory` is itself a **complex operation**.

👉 In Spring, this complexity is handled using a **FactoryBean**.

### FactoryBean Used in Spring JPA

* **`LocalContainerEntityManagerFactoryBean`** is used.

---

## Role of LocalContainerEntityManagerFactoryBean

* Creates the **EntityManagerFactory** object
* Registers it as a **bean definition** in the Spring IOC container

### Required Properties

`LocalContainerEntityManagerFactoryBean` requires:

* `dataSource`
* `jpaProperties`
* `jpaVendorAdapter`

---

## Note on JpaTemplate

---

* In **older Spring versions**, `JpaTemplate` was used for JPA operations.
* From **Spring 6 / Spring Boot 3**, **`JpaTemplate` is completely removed**.

### Recommended Modern Approach

* Use **`EntityManager`**
* Or use **Spring Data JPA repositories**

✅ The shown code follows the **modern and recommended approach**.

---

## `@PersistenceContext`

---

```java
@PersistenceContext
private EntityManager entityManager;
```

### Meaning

* Injects a **JPA `EntityManager`** into the class.

### EntityManager Responsibilities

`EntityManager` is the core JPA interface used to:

* Execute queries
* Persist entities
* Update data
* Delete data

---

## Important Behavior

* Spring injects a **proxy-managed EntityManager**
* It is:

  * Automatically tied to the **current transaction**
  * Automatically **opened and closed**

📌 **Do NOT create `EntityManager` manually** in Spring Boot applications.
