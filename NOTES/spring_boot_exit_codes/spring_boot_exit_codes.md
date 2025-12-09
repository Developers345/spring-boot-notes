# Spring Boot Application Exit

## Exit code (or) Application Exit Code

- Exit code indicates the status of the terminating program — whether it ended successfully or failed during termination. This information is passed to the operating system.  
- Exit code meanings:  
  - `0` → successful execution and termination  
  - `non-zero` → error code indicating failure  
- To check exit codes:
  - Windows → `echo %ErrorLevel%`
  - Linux → `$?`

- **Spring Boot Application Exit Codes**
  - `0` → success  
  - `non-zero` → failure  

## Pictorial representation - 1 
<img width="1338" height="322" alt="Screenshot 2025-12-08 184759" src="https://github.com/user-attachments/assets/b6e6540a-ede0-43e6-8289-dde128922c8d" />

## Pictorial representation - 2 
<img width="1882" height="561" alt="Screenshot 2025-12-08 184856" src="https://github.com/user-attachments/assets/dcb29ec0-ee79-43e8-9b24-d8474c7cb371" />

- Spring Boot allows custom exit codes instead of returning default exit codes when the application terminates.

- Spring Boot is highly suitable for microservice-based applications (Spring Boot + Microservices).

- `System.exit(1)` terminates the application and passes the exit code to the OS or JVM.

---

# Sample Code – Why Spring Boot Provides Custom Exit Codes

```java
@Component
class ShipppingTracker {
  public String track(String awbNo) {
    return "in-transit";
  }
}

class Application {
  public static void main(String[] args) {
    ApplicationContext context = SpringApplication.run(Application.class, args);
    ShipppingTracker shipppingTracker = context.getBean("shipppingTracker", ShipppingTracker.class);
    String status = shipppingTracker.track("a306");
    System.out.println(status);
    System.exit(1); // Don't call System.exit() directly — indicates abnormal termination
  }
}
````

* After the IOC container is created, **Spring Boot automatically registers shutdown hooks**, so during termination it will call the destroy methods on all bean definitions.
  Internally Spring runs:

  ```java
  ((ConfigurableApplicationContext) context).registerShutdownHook();
  ```

* If `System.exit(12)` is called directly:

  * It is considered **abnormal termination**, even though lifecycle methods may still run due to the shutdown hook.
  * Why abnormal?

    1. IOC container is not explicitly closed before exit.
    2. SpringApplication tasks like event publishing and proper container shutdown may not execute correctly.

Therefore, **avoid calling `System.exit()` directly**.

---

# How to Call System.exit() Properly in Spring Boot Application

```java
class Application {
  public static void main(String[] args) {
    ApplicationContext context = SpringApplication.run(Application.class, args);
    try {
      ShipppingTracker shipppingTracker = context.getBean("shipppingTracker", ShipppingTracker.class);
      String status = shipppingTracker.track("a306");
      System.out.println(status);
    } finally {
      System.exit(SpringApplication.exit(context));
      // SpringApplication.exit() returns an appropriate exit code
    }
  }
}
```

---

# How to Generate Custom Exit Code on Successful Termination

```java
@Component
class ApplicationExitCodeGenerator implements ExitCodeGenerator {
  public int getExitCode() {
    return 100;
  }
}
```

### Behavior of `SpringApplication.exit(context)`:
 
 - `SpringApplication.exit(context)`  
  When called, it will go to the IOC container and get all bean definitions of the type `ExitCodeGenerator`.

  1. **If no `ExitCodeGenerator` bean definitions exist:**  
     It simply proceeds to close the IOC container, and upon successful close, it returns **0**.

  2. **If `ExitCodeGenerator` beans are found:**  
     The `SpringApplication.exit()` method will invoke `getExitCode()` on each bean definition, collect the **highest negative number** (if one exists), then close the IOC container and return that exit code.
---

# Why Multiple ExitCodeGenerators?

You may want different exit codes based on different success conditions:

* If a database table contains expected data → return exit code A
* If a file was created successfully → return exit code B

Multiple generators allow Spring Boot to evaluate all such conditions and return an appropriate exit code.


# Example Program for Successful Termination of Spring Boot Application Exit Code
---

## ShipmentTracker.java
```java
@Component
public class ShipmentTracker {

    @PostConstruct
    public void init() {
        System.out.println("init() method called");
    }

    public String track(String awbNo) {
        return "in-transit";
    }

    @PreDestroy
    public void close() {
        System.out.println("close() method called");
    }
}
````

---

## BootExitCodesApplication.java

```java
@SpringBootApplication
public class BootExitCodesApplication {

    /* ExitCodeGenerator exitCodeGenerator = new ExitCodeGenerator() {
        @Override
        public int getExitCode() {
            return 100;
        }
    }; */

    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
        return () -> {
            return 100;
        };
    }

    public static void main(String[] args) {

        ApplicationContext context = SpringApplication.run(BootExitCodesApplication.class, args);
        try {
            ShipmentTracker shipmentTracker = context.getBean("shipmentTracker", ShipmentTracker.class);
            String status = shipmentTracker.track("a123");
            System.out.println("Status " + status);

        } finally {
            System.exit(SpringApplication.exit(context));
        }

        // System.exit(12);  Don't call System.exit() directly — it indicates an abnormal termination
    }
}
```

# ExitCodeExceptionMapper

## How to Customize the Exit Code of a Spring Boot Application in Case of Exceptions

There are **2 stages** where exceptions may arise in a Spring Boot application:

1. **During the bootstrapping** of the Spring Boot application.  
2. **During the execution** of the Spring Boot application.

---

## Internal Steps of `SpringApplication.run()`

```java
ApplicationContext context = SpringApplication.run(Application.class, args);
````

SpringApplication internally performs the following steps:

1. Create the empty **Environment** object
2. Detect and load external configuration into the Environment
3. Print the banner
4. Detect the application type and instantiate the appropriate IOC container
5. Load and register Spring factories
6. Invoke `ApplicationContextInitializer`
7. `prepareContext()`
8. `refreshContext()`
9. Execute **CommandLineRunner** and **ApplicationRunner**

   * (IOC container created and bean instantiation completed — only returning context is pending)
   * If an exception occurs here, the IOC container terminates abnormally.

### Bootstrapping Failure

If any exception occurs during startup, Spring Boot must return an exit code to indicate failure.
If application initialization fails, Spring throws an exception.

If:

```java
ApplicationContext context = SpringApplication.run(Application.class, args);
```

fails due to an exception, Spring returns a **random exit code**.

---

## CommandLineRunner – Example Throwing Exception

```java
@Bean
public CommandLineRunner commandLineRunner() {
    return (args) -> {
        throw new UnKnowException("Initialization Failed");
        // Could be InitializationFailedException, ConfigurationErrorException, etc.
    };
}
```

---

## How Spring Wraps Exceptions

Spring wraps the original exception inside a `Throwable` object:

```java
t = new Throwable(e);  // e = original exception
```

Then Spring passes this wrapped `Throwable` to:

```java
ExitCodeExceptionMapper.getExitCode(Throwable exception)
```

---

## Exit Code Event

Spring Boot publishes **ExitCodeEvent** before returning the final exit code.

```java
@Bean
public ApplicationListener<ExitCodeEvent> exitCodeEventApplicationListener() {
    return (e) -> {
        System.out.println("Exit Code " + e.getExitCode());
    };
}
```

---

# Full Example for Spring Boot Exit Code

## ShipmentTracker.java

```java
@Component
public class ShipmentTracker {

    @PostConstruct
    public void init() {
        System.out.println("init() method called");
    }

    public String track(String awbNo) {
        return "in-transit";
    }

    @PreDestroy
    public void close() {
        System.out.println("close() method called");
    }
}
```

---

## UnKnowException.java

```java
public class UnKnowException extends RuntimeException {
    public UnKnowException(String message) {
        super(message);
    }

    public UnKnowException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

---

## BootExitCodesApplication.java

```java
@SpringBootApplication
public class BootExitCodesApplication {

    @Bean
    public ExitCodeExceptionMapper exitCodeExceptionMapper() {
        return new ExitCodeExceptionMapper() {
            @Override
            public int getExitCode(Throwable exception) {
                Throwable t = exception.getCause();
                System.out.println("t---" + t);

                if (t instanceof UnKnowException) {
                    return -30;
                }
                return 1;
            }
        };
    }

    @Bean
    public ApplicationListener<ExitCodeEvent> exitCodeEventApplicationListener() {
        return (e) -> {
            System.out.println("Exit Code " + e.getExitCode());
        };
    }

    //@Bean
    public CommandLineRunner commandLineRunner() {
        return (args) -> {
            throw new UnKnowException(
                "Initialization Failed",
                new UnKnowException("Initialization Failed")
            );
        };
    }

    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
        return () -> {
            return 100;
        };
    }

    public static void main(String[] args) {

        ApplicationContext context = SpringApplication.run(BootExitCodesApplication.class, args);

        try {
            ShipmentTracker shipmentTracker =
                context.getBean("shipmentTracker", ShipmentTracker.class);

            String status = shipmentTracker.track("a123");
            System.out.println("Status " + status);

        } finally {
            System.exit(SpringApplication.exit(context));
        }

        // System.exit(12); Don't call System.exit() directly — indicates abnormal termination
    }
}
```

# ExitCodeExceptionMapper

## How to Customize the Exit Code of a Spring Boot Application in Case of Exceptions

There are **2 stages** where exceptions may arise in a Spring Boot application:

1. **During the bootstrapping** of the Spring Boot application.  
2. **During the execution** of the Spring Boot application.

---

## Internal Steps of `SpringApplication.run()`

```java
ApplicationContext context = SpringApplication.run(Application.class, args);
````

SpringApplication internally performs the following steps:

1. Create the empty **Environment** object
2. Detect and load external configuration into the Environment
3. Print the banner
4. Detect the application type and instantiate the appropriate IOC container
5. Load and register Spring factories
6. Invoke `ApplicationContextInitializer`
7. `prepareContext()`
8. `refreshContext()`
9. Execute **CommandLineRunner** and **ApplicationRunner**

   * (IOC container created and bean instantiation completed — only returning context is pending)
   * If an exception occurs here, the IOC container terminates abnormally.

### Bootstrapping Failure

If any exception occurs during startup, Spring Boot must return an exit code to indicate failure.
If application initialization fails, Spring throws an exception.

If:

```java
ApplicationContext context = SpringApplication.run(Application.class, args);
```

fails due to an exception, Spring returns a **random exit code**.

---

## CommandLineRunner – Example Throwing Exception

```java
@Bean
public CommandLineRunner commandLineRunner() {
    return (args) -> {
        throw new UnKnowException("Initialization Failed");
        // Could be InitializationFailedException, ConfigurationErrorException, etc.
    };
}
```

---

## How Spring Wraps Exceptions

Spring wraps the original exception inside a `Throwable` object:

```java
t = new Throwable(e);  // e = original exception
```

Then Spring passes this wrapped `Throwable` to:

```java
ExitCodeExceptionMapper.getExitCode(Throwable exception)
```

---

## Exit Code Event

Spring Boot publishes **ExitCodeEvent** before returning the final exit code.

```java
@Bean
public ApplicationListener<ExitCodeEvent> exitCodeEventApplicationListener() {
    return (e) -> {
        System.out.println("Exit Code " + e.getExitCode());
    };
}
```

---

# Full Example for Spring Boot Exit Code

## ShipmentTracker.java

```java
@Component
public class ShipmentTracker {

    @PostConstruct
    public void init() {
        System.out.println("init() method called");
    }

    public String track(String awbNo) {
        return "in-transit";
    }

    @PreDestroy
    public void close() {
        System.out.println("close() method called");
    }
}
```

---

## UnKnowException.java

```java
public class UnKnowException extends RuntimeException {
    public UnKnowException(String message) {
        super(message);
    }

    public UnKnowException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

---

## BootExitCodesApplication.java

```java
@SpringBootApplication
public class BootExitCodesApplication {

    @Bean
    public ExitCodeExceptionMapper exitCodeExceptionMapper() {
        return new ExitCodeExceptionMapper() {
            @Override
            public int getExitCode(Throwable exception) {
                Throwable t = exception.getCause();
                System.out.println("t---" + t);

                if (t instanceof UnKnowException) {
                    return -30;
                }
                return 1;
            }
        };
    }

    @Bean
    public ApplicationListener<ExitCodeEvent> exitCodeEventApplicationListener() {
        return (e) -> {
            System.out.println("Exit Code " + e.getExitCode());
        };
    }

    //@Bean
    public CommandLineRunner commandLineRunner() {
        return (args) -> {
            throw new UnKnowException(
                "Initialization Failed",
                new UnKnowException("Initialization Failed")
            );
        };
    }

    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
        return () -> {
            return 100;
        };
    }

    public static void main(String[] args) {

        ApplicationContext context = SpringApplication.run(BootExitCodesApplication.class, args);

        try {
            ShipmentTracker shipmentTracker =
                context.getBean("shipmentTracker", ShipmentTracker.class);

            String status = shipmentTracker.track("a123");
            System.out.println("Status " + status);

        } finally {
            System.exit(SpringApplication.exit(context));
        }

        // System.exit(12); Don't call System.exit() directly — indicates abnormal termination
    }
}
```
## ExitCodeGenerator

In a Spring Boot application, we can customize the exit code of our application using `ExitCodeGenerator`.  
When we call `SpringApplication.exit(context)` at the end of the application, it will invoke all `ExitCodeGenerator` beans, gather the exit code of the application, and return it.  
This returned exit code must then be passed to `System.exit(int)`.

---

## ExitCodeExceptionMapper

While bootstrapping a Spring Boot application, if an exception occurs and we want to return an exit code specific to the exception, we use `ExitCodeExceptionMapper` to map an exception to a specific exit code.

When an exception is thrown during startup, the `SpringApplication.run(args)` method receives it.  
Then Spring checks whether any `ExitCodeExceptionMapper` is present, and if so, it calls:

```java
int getExitCode(Throwable t)
````

Inside this method, we need to identify the type of exception and return an appropriate exit code to `SpringApplication.run()`, which eventually terminates the application with that specific exit code.

---

### Notes

* When we use `ExitCodeExceptionMapper`, it means any exception raised inside `SpringApplication.run()` cannot be handled by the developer (because we cannot write a try-catch block around `SpringApplication.run()`), so Spring handles it using the mapper.

* For **application-specific runtime exceptions (custom exceptions)**, we **cannot** use `ExitCodeExceptionMapper`, because the control is with the developer, and we can write exit code logic in the `finally` block.

---

## Example Sample Code for Our Own Exception

```java
@SpringBootApplication
class SpringBootApplication {
   public static void main(String[] args) {
      ApplicationContext context = SpringApplication.run(SpringBootApplication.class, args);

      try {
         File inFile = new File("d:\\redme.txt");
         FileInputStream inFileInputStream = new FileInputStream(inFile);

      } catch (IOException e) {
         e.printStackTrace();
      } finally {
         SpringApplication.exit(context);
         System.exit(0);
      }
   }
}
```
