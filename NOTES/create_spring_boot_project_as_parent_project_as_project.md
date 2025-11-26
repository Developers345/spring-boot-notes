# Multiple ways of creating the Spring Boot project

## Approach - 1 : Parent project as a project

### To Generate Maven project through command prompt

```bash
mvn archetype:generate -DgroupId=boot -DartifactId=boot-init -Dversion=1.0.0-SNAPSHOT \
-DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-quickstart \
-DarchetypeVersion=1.5
````

### Project Structure

```
boot-init
├─ src
│  ├─ main
│  │  ├─ java
│  │  └─ resources
│  └─ test
│     ├─ java
│     └─ resources
└─ pom.xml
```

---

## `pom.xml`

```xml
<project>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.0</version>
    <relativePath/>
  </parent>

  <groupId>boot</groupId>
  <artifactId>boot-init</artifactId>
  <packaging>jar</packaging>
  <version>1.0.0-SNAPSHOT</version>

  <dependencies>
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter</artifactId>
     </dependency>
   </dependencies>
</project>
```

---

## Advantages

1. All dependencies are inherited from the parent, so no need to declare versions in our `pom.xml`.
   Easy to switch the Spring Boot version by updating only the parent version.

2. Spring Boot plugin will be added automatically from the parent.

3. Character encoding = **UTF-8**.

4. `java.version = 17` or higher is compatible with Spring Boot **3.4.0**.

---

## Example Program

### `FuelTank.java`

```java
@Component
public class FuelTank {

    @Value("${type}")
    private String type;

    @Override
    public String toString() {
        return "FuelTank{" +
                "type='" + type + '\'' +
                '}';
    }
}
```

### `Motor.java`

```java
@Component
public class Motor {

    @Value("${motorNo}")
    private String motorNo;

    @Autowired
    private FuelTank fuelTank;

    @Override
    public String toString() {
        return "Motor{" +
                "motorNo='" + motorNo + '\'' +
                ", fuelTank=" + fuelTank +
                '}';
    }
}
```

---

## `application.properties`

*(This is the default properties filename)*

```
motorNo=M123
type=Disel
```

---

## `BootInitApplication.java` (Test class)

```java
@SpringBootApplication
public class BootInitApplication {

    public static void main(String[] args) {

        ApplicationContext context = SpringApplication.run(BootInitApplication.class, args);
        Motor motor = context.getBean("motor", Motor.class);

        System.out.println(motor);
    }
}
```

---

## Complete `pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.0</version>
    <relativePath/>
  </parent>

  <groupId>boot</groupId>
  <artifactId>boot-init</artifactId>
  <packaging>jar</packaging>
  <version>1.0.0-SNAPSHOT</version>

  <name>boot-init</name>

  <properties>
    <java.version>17</java.version>
  </properties>

  <dependencies>
     <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter</artifactId>
     </dependency>
   </dependencies>

  <build>
  </build>

</project>
```

---

## `JavConfig.java`

```java
@Configuration
@ComponentScan(basePackages = {"com.boot.init.*"})
@PropertySource("classpath:application.properties")
public class JavConfig {}
```

---

## Important Note

Instead of writing a Java configuration class, the Spring Boot main class (`BootInitApplication`) itself acts as the configuration class.
So **no need for a separate Java configuration class**.

---

## `@SpringBootApplication`

`@SpringBootApplication` internally enables:

1. `@Configuration`
2. `@ComponentScan(basePackages = {"com.boot.init.*"})`
3. `@EnableAutoConfiguration`

If classes are created inside the base package, Spring Boot automatically scans them and loads `application.properties` without requiring `@PropertySource`.

