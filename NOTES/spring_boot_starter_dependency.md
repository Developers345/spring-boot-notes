# Starter Dependencies Definition

- There are a bunch of starter dependencies that are already created and provided by Spring Boot. Depending on the type of application that we are developing, we just need to add the respective starter dependency to our project so that all module dependencies and third-party libraries are included automatically. This way, we never end up identifying and searching for versions and compatibility of third-party libraries used in the project.

- Every Spring Boot starter dependency starts with **`spring-boot-starter-*`**, where `*` indicates modules such as:
  - `spring-boot-starter-web`
  - `spring-boot-starter-jdbc`


- Even though we can work with Spring Boot without a build tool, it is highly recommended to build Spring Boot applications using a build system that supports dependency management such as:  
  i. Maven  
  ii. Gradle  
  iii. Ant + Ivy  

- If we don't use a build tool, then we need to manually download Spring Boot library dependencies and add them to the classpath of our application. This takes a lot of setup time. Instead, it is better to use a build tool.

- Even when using a build tool like Maven, we still need to configure dependencies for our project by writing dependency configurations in `pom.xml` or `build.gradle` (in case of Gradle).

- Let’s say we are using the Maven build tool for developing a project. What should we do to set up the project?  
  i. Run Maven to create the project.  
  ii. Configure dependencies and plugins in the `pom.xml` of the Maven project.  
     - When configuring dependencies, we need to identify which Spring module depends on what other Spring modules and in turn what third-party libraries they depend on.  
     - This takes a lot of time and effort to identify the correct libraries and their compatible versions.  
     - We might end up troubleshooting library and version mismatches, which reduces developer productivity and increases project setup time.

- To rescue developers from this problem, **Spring Boot provides Starter Dependencies**.  
  Spring Boot Starter Dependencies are nothing but pre-defined Maven dependencies that include **transitive dependencies**.

### `spring-boot-starter` (basic)

Includes:  
- spring-core  
- spring-context  
- spring-context-support  
- spring-beans  
- commons-beans  
- commons-utils  
- commons-logging  
- log4j  

### Artifacts: `spring-boot-starter-*`

```

empty (jars)
└── pom.xml
├── groupId
├── artifactId
├── version
└── dependencies (transitive dependencies)
├── Spring module dependencies
└── Third-party libraries

```

### Examples of Spring Boot Starter Dependencies:
- `spring-boot-starter-web`  
- `spring-boot-starter-jdbc`  
- `spring-boot-starter-datajpa`  
- `spring-boot-starter-jpa`  

---

- Spring Boot gives us a **jump-start experience** in building Spring Framework applications.  
- In core Java or Java web applications, we traditionally download JAR files and place them in the classpath.  
- Instead, we can use build tools like Maven or Gradle to automatically manage dependencies (including transitive dependencies).


# Spring Boot Initializer Tool

Spring Boot provides a powerful and convenient tool known as the **Spring Boot Initializer** (commonly referred to as **Spring Initializr**). This tool is designed to simplify the process of bootstrapping a new Spring Boot project by offering a streamlined and intuitive interface. It eliminates the need to manually configure complex project structures or repeatedly set up boilerplate files.

Spring Boot Initializr can be accessed and utilized in two primary ways:

- **Web Interface**  
  The web-based Spring Initializr (available through a browser) offers a clean, interactive UI where developers can specify project details such as:
  - Project type (Maven or Gradle)
  - Language (Java, Kotlin, or Groovy)
  - Spring Boot version
  - Dependencies (like Web, JPA, Security, etc.)
  
  This approach is ideal for quickly generating project structure files and downloading them as a ZIP package to begin development immediately.

- **IDE Integration**  
  Most modern IDEs such as IntelliJ IDEA, Spring Tools Suite (STS), and Eclipse provide built-in Spring Initializr support. This allows developers to generate new Spring Boot projects directly from within the IDE environment.  
  Using this method, you benefit from:
  - Faster project creation  
  - Automatic IDE configuration  
  - Direct dependency selection  
  - Immediate project import and build setup  

