# How does `SpringApplication.run()` Work in Spring Boot?

In a normal Spring Framework application, we manually create the IoC container:

```java
ApplicationContext context = new AnnotationConfigApplicationContext(JavaConfig.class);
Toy toy = context.getBean("toy", Toy.class);
```

But in **Spring Boot**, the developer **does not create the IoC container manually**.
Spring Boot automatically creates and configures it:

```java
ApplicationContext context = SpringApplication.run(Application.class, args);
```

So the question is:

## What Exactly Does `SpringApplication.run()` Do?

The entire Spring Boot ecosystem is built around the `SpringApplication.run()` method.
It performs many startup activities required to run a Spring Boot application.
These are packaged inside the **SpringApplication** class.

Below are the major operations:

---

## 1. Creates an Empty Environment Object

Spring Boot prepares an initial **Environment** that will later hold properties, profiles, system variables, etc.

---

## 2. Loads Application Configuration Into the Environment

It detects and loads configuration properties from various locations:

1. `/config` folder under the project root
2. The current project directory
3. `/config` package inside the application
4. `application.properties` or `application.yaml` under the classpath

Both `application.properties` and `application.yaml` are supported.

---

## 3. Prints the Banner

The Spring Boot banner is displayed unless disabled.

---

## 4. Determines Application Type (`webApplicationType`)

Spring Boot inspects the classpath to decide the nature of the application:

### If Spring MVC JAR Exists

→ Treat as **WEB** → Creates
`AnnotationConfigServletWebServerApplicationContext`

### Else If Spring WebFlux JAR Exists

→ Treat as **REACTIVE** → Creates
`AnnotationConfigReactiveWebServerApplicationContext`

### Else

→ Treat as **NON-WEB** → Creates
`AnnotationConfigApplicationContext`

This is how Spring Boot decides which IoC container to instantiate.

---

## 5. Loads Auto-Configuration Classes (SpringFactories)

Every Spring Framework component has a corresponding **AutoConfiguration** class:

Example:
`DriverManagerDataSource` → `DriverManagerDataSourceAutoConfiguration`

Spring Boot uses these auto-config classes to instantiate and configure framework classes automatically.

These are collectively called **SpringFactories classes**.

SpringFactories are then instantiated and registered in the IoC container.

---

## 6. Executes `ApplicationContextInitializer`

After the IoC container is created, Spring Boot invokes the initializer.
Developers can use this to apply additional custom logic to the context.

---

## 7. `prepareContext`

At this stage Spring Boot:

* Reads bean metadata
* Scans annotations
* Loads environment properties
* Prepares the IoC container for refresh

---

## 8. `refreshContext`

Spring Boot now:

* Instantiates beans
* Applies bean definitions
* Considers active profiles
* Performs dependency injection

This is the actual creation of the Spring application structure.

---

## 9. Executes `CommandLineRunner` and `ApplicationRunner`

After the context is fully ready:

* Runs all `CommandLineRunner` beans
* Runs all `ApplicationRunner` beans
* Starts the embedded server if it’s a web app
* Returns the fully initialized **ApplicationContext**

---

## 10. Publishes Startup Events and Executes Listeners

At different phases, Spring Boot publishes startup events such as:

* `ApplicationStartingEvent`
* `ApplicationEnvironmentPreparedEvent`
* `ApplicationPreparedEvent`
* `ApplicationStartedEvent`
* `ApplicationReadyEvent`

Listeners subscribed to these events are executed accordingly.

---

# Summary

`SpringApplication.run()` is the **heart** of Spring Boot.
It:

* Creates and configures the IoC container
* Loads properties
* Determines application type
* Applies auto-configuration
* Refreshes the context
* Executes runners
* Publishes events
* Finally returns the IoC container

This is why Spring Boot apps **must** be started with `SpringApplication.run()`.

---
