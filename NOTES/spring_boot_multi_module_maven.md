# Spring Boot Multi-Module Maven

---

## Example Snippet

### Project Structure

```

rapido
└── pom.xml

````

---

## `pom.xml` (Parent Project)

```xml
<project>

  <parent>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-parent</artifactId>
     <version>3.4.0</version>
  </parent>

  <groupId>rapido</groupId>
  <artifactId></artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <modules>
    <module>rapido-common</module>
    <module>rapido-customer</module>
  </modules>

  <dependencyManagement>
     <dependencies>
         <!-- dependencies to be inherited by child modules -->
         <!-- define Spring Boot starter dependencies here (no versions needed in child) -->
     </dependencies>
  </dependencyManagement>

  <build>
     <pluginManagement>
        <plugins>
           <plugin>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
        </plugins>
     </pluginManagement>
  </build>
</project>
````

---

# rapido-common

## `pom.xml`

```xml
<project>
  <parent>
    <groupId>rapido</groupId>
    <artifactId></artifactId>
    <version>1.0.0-SNAPSHOT</version>
  </parent>

  <artifactId>rapido-common</artifactId>

  <dependencies>
    <!-- define dependencies without version -->
  </dependencies>
</project>
```


