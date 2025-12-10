# CommandLineRunners and ApplicationRunners

* **CommandLineRunners** or **ApplicationRunners** are used to perform startup activities in a Spring Boot application.

## What happens inside `SpringApplication.run(Application.class, args)`?

1. Create an empty **Environment** object
2. Detect and load external configurations into the environment
3. Print banner
4. Identify the application type and instantiate the corresponding IoC container
5. Instantiate and register the Spring factories
6. Invoke the `ApplicationContextInitializer`
7. `prepareContext()`
8. `refreshContext()` — all bean definitions in the IoC container are fully instantiated

> **Now we want to perform startup activity before the user starts accessing the application.**

9. The `SpringApplication` class now fetches all bean definitions of type **CommandLineRunner** / **ApplicationRunner** from the refreshed IoC container and calls their `run()` method to perform startup tasks.

---

## Example Code

```java
ApplicationContext context = SpringApplication.run(Application.class, args);

@Component
class ApplicationCommandLineRunner implements CommandLineRunner {
    public void run(String[] args) {
        // startup logic
    }
}

class MyApplicationRunner implements ApplicationRunner {
    public void run(ApplicationArguments args) {
        // startup logic
    }
}
```

### Running with Arguments

```bash
java -jar boot-cmd-line-runners.jar 10 20 -log=debug
```

* `10`, `20` → **non-option arguments**
* `-log=debug` → **option argument**

---

# Example 1

### Servlet Applications

When a web application (Servlet-based) is deployed, if you want to perform some initialization **before servlet objects are instantiated**, you use a **ServletContextListener**.

---

# Example 2: Spring Web MVC Flow

## Flow of Spring MVC

* After deployment, the servlet container calls the **DispatcherServlet** through its `service()` method.
* When the first request comes to the DispatcherServlet:

  * It creates the IoC container in its `init()` method.
  * DispatcherServlet then calls controllers based on mappings.

---

## Why `ServletContextListener` cannot be used in Spring MVC initialization?

### **Reason 1**

`ServletContextListener` belongs to **Servlet API**, not Spring MVC.
It is not a Spring-managed component.

### **Reason 2**

`ServletContextListener` runs **after application deployment but before servlet instantiation**.
But in Spring MVC:

* DispatcherServlet creates the IoC container **Servlet calls the Dispatcher Servlet**.
* Initialization activities you want to perform after IoC container creation and bean definition as well  cannot be done with `ServletContextListener`.

👉 Therefore, we use **CommandLineRunners / ApplicationRunners** in Spring Boot for such initialization.

---
