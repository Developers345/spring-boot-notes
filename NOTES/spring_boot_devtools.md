# Spring Boot DevTools
---------------------

-> **DevTools** means providing development-related tools.  
The DevTools module includes features that help developers be more productive while developing applications.

## How to register DevTools in our project?

Add the following dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
````

Adding the `spring-boot-devtools` dependency to your project enables DevTools features automatically.
We do not need to configure anything explicitly.

⚠️ **Caution**
All the features provided by DevTools are related to the development phase and **should not be used in production environments**.

### Maven Build Configuration

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>      
    </plugin>
  </plugins>
</build>
```

-> By default, DevTools will be **excluded from JAR/WAR packaging** by the `spring-boot-maven-plugin`, ensuring DevTools is not available in production environments.

### Note

-> When working in a development environment, there is **no need to package the application as JAR/WAR**.
Simply run the application from the `main()` method in Spring Boot.

---

## What are the features provided by DevTools?

### 1. Property Defaults

-> Template engine example: **Thymeleaf**
The purpose of Thymeleaf is to display dynamic data in views.

* JSP pages require compilation and translation, which takes more time and can impact performance.
* HTML pages represent only static data.

-> To overcome these limitations, **Thymeleaf** comes into the picture.

-> Thymeleaf helps in building **dynamic views at runtime** with dynamic data, which can be rendered to the end user.

-> To optimize performance, Thymeleaf enables **template caching by default**.

However, during development we frequently modify templates, and changes may not reflect due to caching.
We can disable caching manually using:

```properties
spring.thymeleaf.cache=false
```

👉 If DevTools is added to the project:

* DevTools automatically configures this property to `false` in the **development environment**
* Caching will be enabled automatically in **production**

---
### Pictorial Representation
<img width="1894" height="398" alt="Screenshot 2026-01-11 143131" src="https://github.com/user-attachments/assets/5dac7e30-d464-46c4-86f1-1692abbe5494" />

### 2. Automatic Restarts of the Application

#### How does DevTools identify modified classes?

-> DevTools uses **special class loaders** that identify modified classes based on the **timestamp** of the class files.

-> Any changes in the application are detected and reloaded into JVM memory, replacing the older versions of the classes.

-> This eliminates the need for **manual build, deployment, and restart** of the application.

-> DevTools supports **Hot Code Replacement**.

#### Note

-> If multiple files are modified or the same file is modified multiple times, and DevTools is unable to replace the bytecode in JVM memory:

* DevTools automatically performs a **minimal / quick restart** of the application.

👉 **Minimal / Quick Restart** means:

* The web server (Tomcat) is **not restarted**
* The application is removed and re-deployed quickly

---

### 3. LiveReload Server

-> The **LiveReload Server** detects changes in the application and automatically refreshes browser pages.

-> This allows developers to see changes reflected immediately in the browser **without any manual refresh**.

---

### 4. Remote Debugging

-> DevTools supports **remote debugging**, allowing developers to debug applications running in remote environments.

---

### 5. Remote Deployment of Code Changes

-> DevTools also supports **remote deployment**, enabling developers to apply code changes to remote applications efficiently during development.

```
```
