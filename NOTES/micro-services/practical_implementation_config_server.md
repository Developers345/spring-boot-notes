# Config Server
---------------

→ When we package configuration information of a microservice along with  
`application.yml` or `application.properties`, the following problems arise:

### Problems
1. **Environment switching issue**
   - When switching from one environment to another (dev → test → prod),
     we must modify `application.yml/properties` of **each service**.
   - Since configuration is packaged along with the service:
     - All running instances must be stopped
     - Configuration must be modified
     - Services must be re-deployed

2. **Configuration duplication**
   - Some configurations are common across multiple services
   - These configurations get duplicated in every microservice

---

## Solution: Config Server & Config Client

→ To overcome the above problems, **Config Server** and **Config Client** come into the picture.

→ If we use **Config Server**, we only need to restart the **Config Server**.  
There is **no need to re-deploy microservices**.

---

## Config Server – Definition
-----------------------------

→ **Config Server** is a central server where configuration of **all microservices**
is stored across a microservice-based application.

→ Any configuration change can be reflected **quickly**, without re-deploying services.

---

## Key Points
------------

→ All microservice-specific and common configurations are stored in **Config Server**.

→ Configuration file names are written based on:
```

spring.application.name = rapido-mobile-platform

````

→ Configuration files can be stored in:
- Local file system
- Git repository (local or remote)

→ Config Server monitors:
- File changes (local repository)
- Git commit changes (GitHub / Git server)

→ When a change occurs:
- Config Server pulls the latest configuration
- Reloads it into memory
- Config Clients receive the updated configuration

---

## Internals of Config Server & Config Client
--------------------------------------------

→ While creating the **IoC container**, Spring internally creates an **Environment** object.

→ If a **Config Client** is configured:
- Config Client pulls latest configuration from Config Server
- Configuration is loaded into the Environment
- IoC container is created using this updated Environment

---

## How Config Client Pulls Configuration
----------------------------------------

→ Config Clients are configured with the **Config Server URL**.

→ Config Clients use a **pull-based mechanism**:
- Periodically pull latest configuration from Config Server
- Each microservice must include Config Client dependency

---

## Practical Working – Config Client
------------------------------------

### Add Config Client Dependency (user-service)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
````

### Spring Cloud Dependency Management

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2024.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

### `application.yml` (Config Client)

```yaml
server:
  port: 8081

spring:
  application:
    name: user-service
  config:
    import: configserver:http://localhost:8888
```

→ `spring.config.import` tells the microservice **where the Config Server is running**.

---

## Config Server – Spring Boot Project

---

### Project Name

* `rapido-config-server`

### Add Config Server Dependency

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### Spring Cloud Dependency Management

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2024.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

### `application.yml` (Config Server)

```yaml
server:
  port: 8888

spring:
  application:
    name: rapido-config-server
  cloud:
    config:
      server:
        git:
          uri: file://${user.home}/rapido-config-repo
```

→ `spring.cloud.config.server.git.uri` specifies where configuration files are stored:

* Local file system
* Local Git repository
* Remote GitHub repository

---

## Config Server Main Class

---

```java
@SpringBootApplication
@EnableConfigServer
public class RapidoConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(RapidoConfigServerApplication.class, args);
    }
}
```
## Pictorial Representation - Config Server 
<img width="1914" height="732" alt="Screenshot 2026-01-16 130744" src="https://github.com/user-attachments/assets/cff667db-2f53-47a2-901c-0bbe16dd777f" />

## Pictorial Representation - Config Server project setup 
<img width="1862" height="694" alt="Screenshot 2026-01-16 155015" src="https://github.com/user-attachments/assets/e0cddb4f-c812-40bb-9a48-7d6b93638114" />
