# Spring Profiles

- Profiles are used for switching the configuration with which we instantiate the bean definitions pertaining to different environments.  
  **(or)**  
- Profiles are used for switching between one environment to another easily without making changes in the application.

---

# Configuration Externalization

- To avoid hardcoding values within our classes, we keep them outside the Java code so any required changes can quickly be made and reflected for our application components.  
  Profiles, on the other hand, keep multiple copies of configuration values (dev, test, stg, prod) for different environments, allowing easy switching.

- Configuration externalization and Profiles both are different concepts.

- We generally avoid hardcoding configuration parameters inside Java classes and keep them in properties/XML-based configuration files. This helps us manage changes easily.  
  But while switching from one environment to another, we must modify all configuration parameters manually, which leads to problems:

### Problems

1. High chance of modifying something incorrectly in configuration parameters, causing application failure.
2. Switching back and forth becomes tedious because every time the environment changes, we need to modify configurations manually.

- To avoid these problems, **Spring introduced Profiles**.  
  We maintain multiple copies of configuration parameters, each belonging to different environments.  
  At runtime, based on the chosen environment, Spring automatically picks the appropriate configuration, allowing easy switching with minimal effort.

---

# Example: Using Profiles in Spring Framework

```java
@Configuration
@Profile("dev")
class DevConfig {

}

@Configuration
@Profile("test")
class TestConfig {

}

@Configuration
@Import({ DevConfig.class, TestConfig.class })
class RootConfig {

}
````

---

# Test Class Example

```java
ApplicationContext context = new AnnotationConfigApplicationContext();

((AnnotationConfigApplicationContext) context)
        .getEnvironment()
        .setActiveProfiles("qa");

((AnnotationConfigApplicationContext) context)
        .register(DevConfig.class, QAConfig.class);

((AnnotationConfigApplicationContext) context).refresh();
```

# Spring Boot Profiles

## Order for Passing Configuration Values
1. Command line (Highest priority)  
2. `application.properties` / `application.yml` (Second priority)  
3. Programmatically (Least priority)

---

# 1. Example Program for Spring Boot Profiles with Custom Component Classes

## Passport.java
```java
@Component
@ConfigurationProperties(prefix = "passport")
public class Passport {

    protected String id;
    protected String passportHolderName;
    protected String issuedBy;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getPassportHolderName() {
        return passportHolderName;
    }

    public void setPassportHolderName(String passportHolderName) {
        this.passportHolderName = passportHolderName;
    }

    public String getIssuedBy() {
        return issuedBy;
    }

    public void setIssuedBy(String issuedBy) {
        this.issuedBy = issuedBy;
    }

    @Override
    public String toString() {
        return "Passport{" +
                "id='" + id + '\'' +
                ", passportHolderName='" + passportHolderName + '\'' +
                ", issuedBy='" + issuedBy + '\'' +
                '}';
    }
}
````

---

## CommonPropertySourceFactory.java

```java
public class CommonPropertySourceFactory implements PropertySourceFactory {

    @Override
    public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {
        YamlPropertiesFactoryBean yamlPropertiesFactoryBean = new YamlPropertiesFactoryBean();
        yamlPropertiesFactoryBean.setResources(resource.getResource());
        
        Properties properties = yamlPropertiesFactoryBean.getObject();
        return new PropertiesPropertySource("custom", properties);
    }
}
```

---

## common-dev.yml

```yaml
passport:
  id: a1456
  passportHolderName: Dev
  issuedBy: Govt of india
```

## common-test.yml

```yaml
passport:
  id: a1457
  passportHolderName: Test
  issuedBy: Govt of india
```

---

## application.properties

```
spring.profiles.active=dev
```

---

## DevConfig.java

```java
@Configuration
@Profile("dev")
@PropertySource(value = "classpath:common-dev.yml", factory = CommonPropertySourceFactory.class)
public class DevConfig {
}
```

## TestConfig.java

```java
@Configuration
@Profile("test")
@PropertySource(value = "classpath:common-test.yml", factory = CommonPropertySourceFactory.class)
public class TestConfig {
}
```

---

## BootProfileApplication.java

```java
@SpringBootApplication
//@EnableConfigurationProperties -> no need >= 3.x versions
@Import({ DevConfig.class, TestConfig.class })
public class BootProfileApplication {

    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(BootProfileApplication.class, args);
        Passport passport = context.getBean("passport", Passport.class);
        System.out.println(passport);
    }
}
```

---

# 2. Example Program for Spring Boot Supported Profiles

## Product.java

```java
@Component
@ConfigurationProperties(prefix = "product")
public class Product {

    protected int productNo;
    protected String productName;

    public int getProductNo() {
        return productNo;
    }

    public void setProductNo(int productNo) {
        this.productNo = productNo;
    }

    public String getProductName() {
        return productName;
    }

    public void setProductName(String productName) {
        this.productName = productName;
    }

    @Override
    public String toString() {
        return "Product{" +
                "productNo=" + productNo +
                ", productName='" + productName + '\'' +
                '}';
    }
}
```

---

## application1.properties (Master Property File)

```
spring.profiles.active=test
```

---

## application-dev.properties

```
product.productNo=100
product.productName=Fridge
```

## application-test.properties

```
product.productNo=101
product.productName=SmartTV
```

---

## application.yml

```yaml
spring:
  profiles:
    active: dev

---
spring:
  config:
    activate:
      on-profile: dev

product:
  productNo: 103
  productName: AC

---
spring:
  config:
    activate:
      on-profile: test

product:
  productNo: 104
  productName: Washing Machine
```

---

## BootProfilesApplication.java

```java
@SpringBootApplication
public class BootProfilesApplication {

    public static void main(String[] args) {
        /*
        SpringApplication springApplication = new SpringApplicationBuilder(BootProfilesApplication.class)
                .profiles("test").build();
        ApplicationContext context = springApplication.run(args);
        */
        
        ApplicationContext context = SpringApplication.run(BootProfilesApplication.class, args);
        Product product = context.getBean("product", Product.class);
        System.out.println(product);
    }
}
```

---

# Note

* Spring Boot first checks the master property/YAML file (`application.properties` / `application.yml`).
* Based on the value of `spring.profiles.active=test`, Spring Boot looks for `application-test.properties` or `application-test.yml`.
* It then activates the profile and loads the corresponding configuration values.
