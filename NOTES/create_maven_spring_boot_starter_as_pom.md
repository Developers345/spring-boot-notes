# Approach - 2 : Parent project as a POM  
### (Without `spring-boot-starter-parent` defined as parent dependency)

---

## When do you use this approach?

- In some organizations, there is a **common Maven parent project** that already defines shared libraries, plugins, and repositories for all internal applications.  
  In such cases, we **cannot** set `spring-boot-starter-parent` as the parent in our Spring Boot project.
- Instead, we **import** Spring Boot dependencies using `<dependencyManagement>` with `type: pom` and `scope: import`.

---

## Example 1 – Snippet

### `pom.xml`

```xml
<project>
 <groupId/>
 <artifactId/>
 <version/>

  <dependencyManagement>
      <dependencies>
         <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.3.4.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
         </dependency>
      </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter</artifactId>
    </dependency>
  </dependencies>
</project>
````

---

## Explanation

* Here, `spring-boot-starter-parent` is added under `dependencyManagement` with **scope: import** and **type: pom**.
* This tells Maven to **import all dependency versions** defined in the Spring Boot parent POM.
* So, you can add any Spring Boot starter dependency **without specifying version numbers**.
* However, **you do NOT get**:

  * plugin configurations
  * Java compiler settings
  * charset encoding

These **must be configured manually**, as shown below.

---

## Build Plugins

```xml
<build>
  <plugins>
    <!-- spring boot plugin -->
    <plugin>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-plugin</artifactId>
       <version>2.3.4.RELEASE</version>
       <executions>
          <execution>
            <phases>
               <goal>repackage</goal>
               <phase>package</phase>
            </phases>
          </execution>
       </executions>
    </plugin>
  </plugins>
</build>
```

### Notes

* Plugins are building blocks in Maven and run at different lifecycle phases.
* Default lifecycle example:

```
package → spring-boot-maven-plugin (goal: repackage)
```

---

# Final Example – Spring Boot Parent Imported as POM

---

## Toy.java

```java
@Component
public class Toy {

    public void play() {
        System.out.println("Playing");
    }
}
```

---

## SpringBootPomApplication.java (Test)

```java
//@SpringBootApplication
@Configuration
@EnableAutoConfiguration
@ComponentScan(basePackages = {"com.boot.pom.*"})
public class SpringBootPomApplication {

    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringBootPomApplication.class,args);
        Toy toy = context.getBean("toy", Toy.class);
        toy.play();
    }
}
```

---

## pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
           http://maven.apache.org/POM/4.0.0 
           http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>boot</groupId>
  <artifactId>boot-pom</artifactId>
  <packaging>jar</packaging>
  <version>1.0.0-SNAPSHOT</version>

  <name>boot-pom</name>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
  </properties>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.0</version>
        <scope>import</scope>
        <type>pom</type>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>3.4.0</version>
        <executions>
          <execution>
            <goals>
              <goal>repackage</goal>
            </goals>
            <phase>package</phase>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

---

## Example 2 – Snippet

```
rapido
 └── pom.xml
```

```xml
<dependencyManagement>
   <groupId>rapido.com</groupId>
   <artifactId>rapido</artifactId>
   <version>1.0.0-SNAPSHOT</version>
   <scope>import</scope>
   <type>pom</type>
</dependencyManagement>
```
