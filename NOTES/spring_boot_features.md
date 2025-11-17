# What is Spring Boot

- Every module of the Spring Framework is used for addressing the functional aspects of developing the application.  
  Unlike the modules of the Spring Framework, **Spring Boot is not meant for building the functional aspects** of the application; it is used for addressing **non-functional requirements** of the application.

- Using Spring Boot we **cannot directly develop the application**, but we can **use Spring Framework modules easily** with reduced configuration effort.

---

## Spring Framework Modules (Functional Aspects)

1. **Spring Core** → Dependency Management  
2. **Spring JDBC** → Database Application  
3. **Spring MVC** → Web Applications  

---

## Purpose of Spring Boot

- Spring Boot is used for addressing the **non-functional aspects** of developing Spring Framework applications.
- It helps in building **fast-paced applications** compared to using Spring Framework alone.
- Spring Boot **configures Spring Framework components using an opinionated approach**  
  (i.e., based on what libraries are present in the classpath, Spring Boot assumes the required modules and configures them, e.g., Spring JDBC).
- If the requirements diverge from default configurations, we can **easily override and customize** them with minimal effort.

Here is your content in **Markdown (.md)** format:

# Features of Spring Boot

## 1. Auto-Configuration (Definition)

Spring Boot provides **Auto-Configuration**, which enables framework components to be automatically configured as **bean definitions** inside the **IOC container** using an **opinionated (default) approach**.  
This means the developer **does not need to manually configure** these components in their application.

If there are **divergent or custom requirements**, developers can **override the defaults with minimal effort**, or **disable specific auto-configurations** when needed.

# Why Auto-Configuration Was Introduced in Spring Boot

- When working with **Spring Framework–based applications**, developers were required to write a **large amount of configuration** related to application components and Spring Framework components inside **Spring Bean Configuration files**.  
  Although Spring provided powerful features, the **time and effort spent** on configuration became a **major challenge**.

- To reduce configuration overhead, Spring introduced **annotations**, but they came with limitations — configuration using annotations is possible **only when the source code is available**.  
  As an alternative, **Java configuration classes**, **programmatic configuration**, and **configurer adapters** were introduced.  
  However, none of these approaches were fully successful in **eliminating configuration complexity**.

- To overcome these issues, **Spring Boot introduced Auto-Configuration**, a feature where most **framework components and third-party libraries** included in the application **automatically get configured into the IOC container**.  
  This eliminates the need for developers to manually configure Spring Framework components.

- If the **default auto-configuration** does not match the application requirements, developers can **fine-tune the configuration with minimal effort**.  
  By simply providing the required **property values**, Spring Boot reads and configures them — **no manual bean configuration is required**.

- Spring Boot also allows developers to **disable auto-configuration classes** and **provide custom auto-configuration implementations** when needed.

