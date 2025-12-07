# Application Properties And Configuration Properties

## YAML = “Ain’t Markup Language”

- YAML allows two types of data structures:  
  1. **Dictionaries** (key–value pairs)  
  2. **Lists**
- Every YAML document starts with `---` and ends with `...`.
- A single YAML file can contain multiple documents.

## application.yml (syntax)

```

---

## ...

## ...

...

````

---

## Sample Example

### application.yml

```yaml
---
id: 10
propertyType: Appartment
propertyName: Ganesh Nilayam 
location: Hyderabad 
price: 2500000
amenities:
  - car parking
  - lift
  - club house 
  - gym
address:
  addressLine1: Y Junction
  addressLine2: Balanagar
  city: Hyderabad
...
````

---

## Problem with `@Value` Annotation

When using Spring Framework, we typically inject primitive values using `@Value`.
But if a class has many attributes (e.g., 30), writing `@Value` for each is difficult.

### Example

```java
@Component
class Property {

  @Value("${id}")
  String id;

  @Value("${propertyType}")
  String propertyType;

  @Value("${propertyName}")
  String propertyName;

  @Value("${location}")
  String location;

  @Value("${price}")
  double price;

  // accessors
}
```

### application.properties

```
id=100
propertyType=Appartment
propertyName=Ganesh Nilayam
location=Hyderabad
price=2500000
```

### Main Application

```java
@SpringBootApplication
class ConfigurationApplication {
  public static void main(String[] args) {
    ApplicationContext context =
        SpringApplication.run(ConfigurationApplication.class, args);
    Property property = context.getBean("property", Property.class);
  }
}
```

---

## BeanPostProcessor

* If you want to perform post-processing logic on every bean before the container returns it, use **BeanPostProcessor**.
* To resolve the `@Value` annotation issue, Spring Boot provides a built-in BeanPostProcessor:

### **ConfigurationPropertiesBindingPostProcessor**

To enable it, add the dependency:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-configuration-processor</artifactId>
</dependency>
```

### Using `@ConfigurationProperties`

```java
@Component
@ConfigurationProperties(prefix = "redbricks")
class Property {

  String id;
  String propertyType;
  String propertyName;
  String location;
  double price;

  // accessors
}
```

### application.properties

```
redbricks.id=100
redbricks.propertyType=Appartment
redbricks.propertyName=Ganesh Nilayam
redbricks.location=Hyderabad
redbricks.price=2500000
```

---

## Enabling Configuration Properties

```java
@EnableConfigurationProperties
@SpringBootApplication
class ConfigurationApplication {
  public static void main(String[] args) {
    ApplicationContext context =
        SpringApplication.run(ConfigurationApplication.class, args);
    Property property = context.getBean("property", Property.class);
  }
}
```

---

## `@EnableConfigurationProperties`

* This annotation enables the required BeanPostProcessor in **Java Config** approach.
* In XML configuration, the equivalent tags are:

```
<context:annotation-config>
```

or

```
<context:component-scan>
```

Yes — **Spring Boot 3.4 introduced an important improvement** related to `@EnableConfigurationProperties`, and that is why your properties are loading even without explicitly using this annotation.

---

# ✅ **What Changed in Spring Boot 3.4?**

### **Before Spring Boot 3.4**

To load `@ConfigurationProperties` classes, you needed one of these:

1. `@EnableConfigurationProperties(MyProps.class)`
2. `@ConfigurationPropertiesScan`
3. A Spring bean annotated with `@ConfigurationProperties`
4. Registering the class manually in configuration

If you **only annotated the class with `@ConfigurationProperties`**, but did *not* use `@EnableConfigurationProperties`, Spring would NOT bind the values (unless it became a bean through some other mechanism).

---

# ✅ **Spring Boot 3.4 Behavior**

Starting with **Spring Boot 3.4**, a major enhancement was added:

> **Any class annotated with `@ConfigurationProperties` that is also a Spring bean will be automatically bound — even without `@EnableConfigurationProperties`.**

➡️ Meaning:
If your `@ConfigurationProperties` class is annotated with `@Component`, `@Configuration`, or is created via `@Bean`, **you no longer need `@EnableConfigurationProperties`.**

---

# 📌 **Example — Works Without `@EnableConfigurationProperties` in Boot 3.4+**

```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private String version;
}
```

Even without:

```java
@EnableConfigurationProperties(AppProperties.class)
```

➡️ **It still loads and binds successfully.**

---

# 📌 Why Does It Work Now?

Because Spring Boot 3.4 introduced:

### 🔹 **Automatic detection and binding of `@ConfigurationProperties` beans**

Spring Boot now checks:

* Is this class annotated with `@ConfigurationProperties`?
* Is it a bean in the ApplicationContext?

If **yes**, it binds it automatically — no manual enabling required.

---

# 🚨 When Do You Still Need `@EnableConfigurationProperties`?

You only need it when:

### ✔️ **The properties class is *not* a bean by default**

Example:

```java
@ConfigurationProperties(prefix = "external")
public class ExternalProps {
    // No @Component here
}
```

Then you must enable it:

```java
@EnableConfigurationProperties(ExternalProps.class)
```

---

# ✅ Summary

| Spring Boot Version | When Properties Bind                                                                 |
| ------------------- | ------------------------------------------------------------------------------------ |
| **≤ 3.3**           | Only when registered via `@EnableConfigurationProperties`, `@Component`, or scanning |
| **≥ 3.4**           | If the class is a Spring bean + has `@ConfigurationProperties`, binding is automatic |

---

# ⭐ Final Answer

**Yes**, Spring Boot 3.4 introduced automatic binding of `@ConfigurationProperties` bean classes.
So even without using `@EnableConfigurationProperties`, your properties load correctly.

---

# Internal of BeanPostProcessor

```java
class ConfigurationPropertiesBindingPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object obj, String beanName) {
        /*
        IOC Container calls this BeanPostProcessor class.
        It checks whether @ConfigurationProperties annotation is present
        for that bean definition.

        - If present → it loads the primitive values from the Environment object
        - If not present → it skips processing
        */
        return obj;
    }

    public Object postProcessAfterInitialization(Object obj, String beanName) {
        return obj;
    }
}
````

* This class (`ConfigurationPropertiesBindingPostProcessor`) is invoked by the IOC Container for **every bean definition** because we enabled the BeanPostProcessor using `@EnableConfigurationProperties`.

---

# `@ConfigurationProperties`

* Only the bean where you annotate with `@ConfigurationProperties` will be injected with values.
* Remaining beans **will not** be processed.
* IOC Container calls the BeanPostProcessor, which checks whether the bean has `@ConfigurationProperties`.

  * If **present**, values are loaded into the environment and then injected.
  * If **not present**, it is skipped.

---

# Full Example for Spring Boot Configuration Properties

## Property.java

```java
@Component
@ConfigurationProperties(prefix = "redbricks")
public class Property {

    protected String id;
    protected String propertyType;
    protected String propertyName;
    protected String location;
    protected double price;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getPropertyType() {
        return propertyType;
    }

    public void setPropertyType(String propertyType) {
        this.propertyType = propertyType;
    }

    public String getPropertyName() {
        return propertyName;
    }

    public void setPropertyName(String propertyName) {
        this.propertyName = propertyName;
    }

    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    @Override
    public String toString() {
        return "Property{" +
                "id='" + id + '\'' +
                ", propertyType='" + propertyType + '\'' +
                ", propertyName='" + propertyName + '\'' +
                ", location='" + location + '\'' +
                ", price=" + price +
                '}';
    }
}
```

---

## application.properties

```
id=100
propertyType=Appartment
propertyName=Ganesh Nilayam
location=Hyderabad
price=2500000
```

---

## application.yml

```yaml
redbricks:
  id: 101
  propertyType: Apparments
  propertyName: Vamsee Nilayam
  location: Hyderabad
  price: 3500000
```

---

# SpringBootConfigurationPropertiesApplication.java

```java
@SpringBootApplication
//@EnableConfigurationProperties 
/* From Spring Boot 3.x onwards, no need to explicitly write this annotation.
   SpringApplication automatically detects @ConfigurationProperties 
   and loads values into the Environment. */
public class SpringBootConfigurationPropertiesApplication {

    public static void main(String[] args) {
        ApplicationContext context =
                SpringApplication.run(SpringBootConfigurationPropertiesApplication.class, args);

        Property property = context.getBean("property", Property.class);
        System.out.println(property);

        // Internally uses ConfigurationPropertiesBindingPostProcessor
    }
}
```

# Custom Property File Loading in Spring Boot

- In a Spring Boot application, **application.properties** or **application.yml** is generally used to hold **Spring Boot framework configuration values**.
- For **component-specific configuration values**, it is better to use a **separate custom configuration file** instead of mixing everything into the main Spring Boot configuration file.

---

## Example Program for Custom Property File Loading

### Passenger.java

```java
@Component
@ConfigurationProperties(prefix = "passenger")
public class Passenger {

    private String id;
    private String passengerName;
    private String age;
    private String gender;
    private String mobileNo;

    public String getPassengerName() {
        return passengerName;
    }

    public void setPassengerName(String passengerName) {
        this.passengerName = passengerName;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public String getMobileNo() {
        return mobileNo;
    }

    public void setMobileNo(String mobileNo) {
        this.mobileNo = mobileNo;
    }

    @Override
    public String toString() {
        return "Passenger{" +
                "id='" + id + '\'' +
                ", passengerName='" + passengerName + '\'' +
                ", age='" + age + '\'' +
                ", gender='" + gender + '\'' +
                ", mobileNo='" + mobileNo + '\'' +
                '}';
    }
}
````

---

## common.properties

```
passenger.id=100
passenger.passengerName=John
passenger.age=35
passenger.gender=Male
passenger.mobileNumber=96587456
```

---

## CustomConfigurationApplication.java

```java
@SpringBootApplication
@PropertySource("classpath:common.properties")
public class CustomConfigurationApplication {

    public static void main(String[] args) {

        ApplicationContext context =
                SpringApplication.run(CustomConfigurationApplication.class, args);

        Passenger passenger = context.getBean("passenger", Passenger.class);
        System.out.println(passenger);
    }
}
```

```java
@SpringBootApplication
@PropertySource("classpath:common.yml")
class Application {
    public static void main(String[] agrs) {

        SpringApplication springApplication =
                new SpringApplicationBuilder(Application.class)
                        .initializer(new CustomConfigurationApplicationContextInitializer())
                        .build();

        ApplicationContext context = springApplication.run(args);
        Passenger passenger = context.getBean("passenger", Passenger.class);
    }
}
````

---

## Important Note

* In the above example, **`common.yml` cannot be used** because
  `@PropertySource` **supports only `.properties` files**, not YAML files.
* `application.properties` and `application.yml` are loaded by the **SpringApplication** class, **not** by `@PropertySource`.

---

# How to Work With Custom YAML Files

There are **2 proper approaches** for loading proprietary YAML files:

### **1. `YamlPropertiesResourceLoader` with `ApplicationContextInitializer`**

### **2. `YamlPropertiesFactoryBean` or `YamlMapFactoryBean` with `PropertySourceFactory`**

---

# 1. YamlPropertiesResourceLoader with ApplicationContextInitializer

* It takes a YAML file as a **Resource object**.
* Reads all YAML configuration values.
* Converts them into one or more **PropertySource** objects.
* Returns them so they can be added to the Spring Environment.

### ❓ Why does `YamlPropertiesResourceLoader` return **multiple** PropertySource objects?

Because:

* A single YAML file can contain **multiple YAML documents** separated by `---`.
* **Each YAML document becomes one PropertySource**.

---

## When Should YAML Property Sources Be Loaded?

We must load YAML **after** the Environment is created but **before** the IOC container creates bean definitions.

### ❌ Incorrect Timing Example

```java
ApplicationContext context = SpringApplication.run(Application.class, args);
// Too late to load properties here—the IOC container and beans are already created.
```

---

# Spring Boot Startup Sequence (Important)

```
1. Create empty Environment object  
2. Detect and load external configuration into Environment  
3. Print banner  
4. Identify application type and instantiate IOC container  
5. Identify & register Spring factories  
6. Invoke ApplicationContextInitializer  ← (Best time to load YAML)  
7. preparedContext (ApplicationPreparedEvent published)
     - You *can* load YAML here but it is async → not guaranteed timing  
8. refreshContext (bean creation happens here)
```

---

# Best Approach: Use **ApplicationContextInitializer**

* It runs **after IOC container is created**
* But **before bean instantiation begins**
* Perfect for loading custom YAML files into the Environment

### ✔ Why this is ideal?

Because:

* YAML gets added to Environment **exactly before** Spring starts resolving `@ConfigurationProperties`.
* Ensures custom YAML values are available to all beans.

---

# Custom ApplicationContextInitializer (Approach 1)

## Example Program for CustomConfigurationApplicationContextInitializer

---

## Passenger.java

```java
@Component
@ConfigurationProperties(prefix = "passenger")
public class Passenger {
    private String id;
    private String passengerName;
    private String age;
    private String gender;
    private String mobileNo;

    public String getPassengerName() {
        return passengerName;
    }

    public void setPassengerName(String passengerName) {
        this.passengerName = passengerName;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public String getMobileNo() {
        return mobileNo;
    }

    public void setMobileNo(String mobileNo) {
        this.mobileNo = mobileNo;
    }

    @Override
    public String toString() {
        return "Passenger{" +
                "id='" + id + '\'' +
                ", passengerName='" + passengerName + '\'' +
                ", age='" + age + '\'' +
                ", gender='" + gender + '\'' +
                ", mobileNo='" + mobileNo + '\'' +
                '}';
    }
}
````

---

## CustomConfigurationApplicationContextInitializer.java

```java
public class CustomConfigurationApplicationContextInitializer
        implements ApplicationContextInitializer<ConfigurableApplicationContext> {

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {

        YamlPropertySourceLoader yamlPropertySourceLoader = null;
        List<PropertySource<?>> propertySources = null;
        ConfigurableEnvironment environment = null;

        try {
            yamlPropertySourceLoader = new YamlPropertySourceLoader();
            propertySources = yamlPropertySourceLoader.load(
                    "custom",
                    applicationContext.getResource("classpath:common.yml")
            );

            environment = applicationContext.getEnvironment();

            for (PropertySource<?> propertySource : propertySources) {
                environment.getPropertySources().addLast(propertySource);
            }

        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

---

## common.yml

```yaml
passenger:
  id: P2
  passengerName: Nick
  age: 32
  gender: Male
  mobileNo: 987456
```

---

## CustomConfigurationApplication.java

```java
@SpringBootApplication
@PropertySource(value = "classpath:common.yml")
public class CustomConfigurationApplication {

    public static void main(String[] args) {

        SpringApplication springApplication =
                new SpringApplicationBuilder(CustomConfigurationApplication.class)
                        .initializers(new CustomConfigurationApplicationContextInitializer())
                        .build();

        ApplicationContext context = springApplication.run(args);

        Passenger passenger = context.getBean("passenger", Passenger.class);
        System.out.println(passenger);
    }
}
```

---

# Approach 2: YamlPropertiesFactoryBean or YamlMapFactoryBean with PropertySourceFactory

* From **Spring 4.1**, support for `PropertySourceFactory` was added.
* `FactoryBean` is used when Spring must create another object as a bean definition.

### YamlFactoryBean Types:

| FactoryBean Type              | Description                                                  |
| ----------------------------- | ------------------------------------------------------------ |
| **YamlPropertiesFactoryBean** | Treats keys & values as **String** like a `.properties` file |
| **YamlMapFactoryBean**        | Creates a **Map** allowing different data types              |

---

# Example Program (Approach 2)

---

## Passenger.java

```java
@Component
@ConfigurationProperties(prefix = "passenger")
public class Passenger {
    private String id;
    private String passengerName;
    private String age;
    private String gender;
    private String mobileNo;

    // getters & setters

    @Override
    public String toString() {
        return "Passenger{" +
                "id='" + id + '\'' +
                ", passengerName='" + passengerName + '\'' +
                ", age='" + age + '\'' +
                ", gender='" + gender + '\'' +
                ", mobileNo='" + mobileNo + '\'' +
                '}';
    }
}
```

---

## CustomYamlPropertySourceFactory.java

```java
public class CustomYamlPropertySourceFactory implements PropertySourceFactory {

    @Override
    public PropertySource<?> createPropertySource(String name, EncodedResource resource)
            throws IOException {

        YamlPropertiesFactoryBean yamlPropertiesFactoryBean = new YamlPropertiesFactoryBean();
        yamlPropertiesFactoryBean.setResources(resource.getResource());

        Properties properties = yamlPropertiesFactoryBean.getObject();

        return new PropertiesPropertySource("customYaml", properties);
    }
}
```

---

## common.yml

```yaml
passenger:
  id: P2
  passengerName: Nick
  age: 32
  gender: Male
  mobileNo: 987456
```

---

## CustomConfigurationApplication.java

```java
@SpringBootApplication
@PropertySource(
        value = "classpath:common.yml",
        factory = CustomYamlPropertySourceFactory.class
)
public class CustomConfigurationApplication {

    public static void main(String[] args) {

        ApplicationContext context =
                SpringApplication.run(CustomConfigurationApplication.class, args);

        Passenger passenger = context.getBean("passenger", Passenger.class);
        System.out.println(passenger);
    }
}
```

Here is your text in clean **Markdown (.md)** format:

---

## Note
- **ApplicationContextInitializer** is used **only for Spring Boot applications**.
- **PropertySourceFactory** can be used in **both Spring Core** and **Spring Boot** applications.
```


