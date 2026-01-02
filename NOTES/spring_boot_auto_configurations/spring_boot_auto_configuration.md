# Spring Boot Autoconfigurations

---

## What are Auto-configurations?

- Auto-configurations are the classes provided by **Spring Boot**. These detect the dependencies available on the **classpath** of the application and configure the framework components automatically, without requiring the developer to configure them manually.

- The idea is that framework classes are developed by **Spring Framework developers**, who already know what configurations are required for those components. Instead of asking application developers to configure everything, the framework configures itself, thereby reducing development time and unnecessary configuration effort.

- When auto-configurations are in place, the framework appears to **self-tune itself** according to the environment in which it is running.

---

## Auto-configuration and Starter Dependencies

- While working with Spring Boot, we add dependencies using `spring-boot-starter-*`.
- Auto-configurations work based on the dependencies added to the project.
- For every `spring-boot-starter-*` dependency, Spring Boot provides a corresponding auto-configuration internally.

Example:

```

spring-boot-starter-jdbc
spring-boot-starter-jdbc-autoconfigurations

````

(Autoconfiguration classes are added based on the starter dependency.)

---

## Sample Code: Spring JDBC **without** Spring Boot

### BikeBo.java

```java
public class BikeBo {

    protected int bikeNo;
    protected String bikeName;
    protected String bikeColor;
    protected String bikeModel;

    public int getBikeNo() { return bikeNo; }
    public void setBikeNo(int bikeNo) { this.bikeNo = bikeNo; }

    public String getBikeName() { return bikeName; }
    public void setBikeName(String bikeName) { this.bikeName = bikeName; }

    public String getBikeColor() { return bikeColor; }
    public void setBikeColor(String bikeColor) { this.bikeColor = bikeColor; }

    public String getBikeModel() { return bikeModel; }
    public void setBikeModel(String bikeModel) { this.bikeModel = bikeModel; }

    @Override
    public String toString() {
        return "BikeBo{" +
                "bikeNo=" + bikeNo +
                ", bikeName='" + bikeName + '\'' +
                ", bikeColor='" + bikeColor + '\'' +
                ", bikeModel='" + bikeModel + '\'' +
                '}';
    }
}
````

---

### BikeDAO.java

```java
@Repository
public class BikeDAO {

    private final String GET_ALL_BIKES =
            "select bike_no,bike_name,bike_color,bike_model from bike";

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public List<BikeBo> getAllBikes() {
        return jdbcTemplate.query(GET_ALL_BIKES, rowMapper);
    }

    RowMapper<BikeBo> rowMapper = new RowMapper<BikeBo>() {
        @Override
        public BikeBo mapRow(ResultSet rs, int rowNum) throws SQLException {
            BikeBo bikeBo = new BikeBo();
            bikeBo.setBikeNo(rs.getInt(1));
            bikeBo.setBikeName(rs.getString(2));
            bikeBo.setBikeColor(rs.getString(3));
            bikeBo.setBikeModel(rs.getString(4));
            return bikeBo;
        }
    };
}
```

---

### JavaConfig.java (Not Required in Spring Boot)

```java
@Configuration
@PropertySource("classpath:db.properties")
@ComponentScan(basePackages = {"com.spring.jdbc.javaconfig"})
class JavaConfig {

    @Autowired
    private Environment env;

    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(env.getProperty("driverClassName"));
        dataSource.setUrl(env.getProperty("url"));
        dataSource.setUsername(env.getProperty("username"));
        dataSource.setPassword(env.getProperty("password"));
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

---

### db.properties

```properties
driverClassName=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/rapido
username=root
password=root
```

---

### Note

* The `JavaConfig.java` class is **not required** in Spring Boot.
* Auto-configuration classes take care of configuring framework components.
* Developers only need to write business classes like `BikeDAO.java`.

---

## Key Points on Auto-configurations

* Auto-configurations configure framework components as **beans with default values**.
* Developers can override or customize these configurations using properties.
* Auto-configurations work closely with **starter dependencies**.
* Auto-configuration classes are included as **transitive dependencies** via Spring Boot starters.

---

## What are Starter Dependencies?

* Starter dependencies are provided by Spring Boot to **quickly work with Spring modules**.
* Adding a starter automatically pulls in all required module dependencies.

### Definitions

* A starter dependency is an **empty Maven project** that contains:

  * Module dependencies
  * Auto-configurations
  * Transitive dependencies of the module

---

### Example Dependency Tree

```
spring-boot-starter-data-jdbc
  |- spring-jdbc:6.0
  |- spring-core
  |- spring-context
  |- spring-context-support
```

---

## Creating a Custom Starter (Example)

```bash
mvn archetype:generate \
-DgroupId=spring-boot-starter \
-DartifactId=spring-boot-starter-data-jdbc \
-Dversion=1.0.0-SNAPSHOT \
-DarchetypeArtifactId=maven-archetype-quickstart
```

### Project Structure

```
spring-boot-starter-data-jdbc
 ├─ src
 │  ├─ main
 │  │  ├─ java
 │  │  └─ resources
 │  └─ test
 └─ pom.xml
```

```bash
mvn clean install
```

---

## SpringApplication.run() Internals

When `SpringApplication.run(Application.class, args)` is called:

1. Creates an empty Environment object
2. Loads external configuration into the Environment
3. Prints the banner
4. Detects application type and creates the appropriate IOC container
5. Registers Spring factories

```
META-INF/spring/
org.springframework.boot.autoconfigure.AutoConfiguration.imports
com.jimage.gui.autoconfigure.JImageGUIAutoconfiguration
com.jimage.bean.autoconfigure.JImageConverterAutoConfiguration
```

---

## Conditional Annotations

Conditional annotations prevent Spring Boot from configuring **all framework components at once**.

### Types of Conditional Annotations

1. `@ConditionalOnClass`
2. `@ConditionalOnBean`
3. `@ConditionalOnProperty`
4. `@ConditionalOnResource`

---

### 1. @ConditionalOnClass

```java
@ConditionalOnClass(com.mysql.cj.jdbc.Driver.class)
@Configuration
class MySqlDriverManagerDataSourceAutoConfiguration {

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource();
    }
}
```

---

### 2. @ConditionalOnBean

```java
@Configuration
@ConditionalOnBean(name = "dataSource")
class JdbcTemplateAutoConfiguration {

    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }
}
```

---

### 3. @ConditionalOnProperty

```properties
use-mysql=true
```

```java
@Configuration
@ConditionalOnProperty(name = "use-mysql", value = "true")
class MySqlDriverManagerDataSourceAutoConfiguration {

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource();
    }
}
```

---

### 4. @ConditionalOnResource

```properties
mysql-driverClass=
mysql-url=
mysql-username=
mysql-password=
```

```java
@Configuration
@ConditionalOnResource("classpath:mysql-db.properties")
class MySqlDriverManagerDataSourceAutoConfiguration {

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource();
    }
}
```

---

## Wrong Project Structure

```
jimage-boot-starter (packaging=pom)
 ├─ jimage
 └─ jimage-autoconfig
```

❌ Cannot be added as a dependency directly.

---

## Correct Project Structure

```
jimage-boot-parent (packaging=pom)
 ├─ jimage
 ├─ jimage-boot-autoconfig
 └─ jimage-boot-starter
```

```text
web-studio
 └─ pom.xml
    └─ dependency: jimage-boot-starter
```

---

## Final Example for Custom Auto-Configuration

**Local Paths**

```
D:\Core java\springboot\auto-configurations\jimage-boot-parent
D:\Core java\springboot\auto-configurations\web-studio
```

**GitHub References**

* Custom starter and auto-configuration
  [https://github.com/Developers345/practical-custom-spring-boot-starter-auto-configuration.git](https://github.com/Developers345/practical-custom-spring-boot-starter-auto-configuration.git)

* Spring Boot application using custom auto-configuration
  [https://github.com/Developers345/practical-web-studio-uses-custom-auto-configuration.git](https://github.com/Developers345/practical-web-studio-uses-custom-auto-configuration.git)


## Note
---

You are mixing **Spring Boot 2.x style** with a **Spring Boot 3.x application**.

---

## ❌ Root Cause (IMPORTANT)

In **Spring Boot 3.x**, the file:

```

spring.factories

```

is **NO LONGER USED** for auto-configuration loading.

So this file:

```

src/main/resources/spring.factories   ❌

````

and this entry:

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.jimage.gui.autoconfigure.JImageGUIAutoconfiguration,\
com.jimage.bean.autoconfigure.JImageConverterAutoConfiguration
````

👉 is **completely ignored** in Spring Boot 3.x
👉 That’s why the debugger never enters your auto-configuration classes

---

## ✅ Correct Way (Spring Boot 3.x+)

### 1️⃣ Delete or Ignore `spring.factories`

❌ **Do NOT use** `spring.factories` for auto-configuration in Spring Boot 3+

---

### 2️⃣ Create the NEW Required File

📁 **Exact path (must match exactly):**

```
src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

📄 **Content:**

```
com.jimage.gui.autoconfigure.JImageGUIAutoconfiguration
com.jimage.bean.autoconfigure.JImageConverterAutoConfiguration
```

---

## ✅ Summary

* `spring.factories` ❌ → **Deprecated for auto-config in Boot 3.x**
* `AutoConfiguration.imports` ✅ → **Mandatory in Boot 3.x+**
* Incorrect file → Auto-configurations **will not load**
* Correct file → Auto-configurations **load automatically**


