# Customize the Spring Boot Application

The `SpringApplication` class performs various activities while starting a Spring Boot application, such as loading external configuration, generating the banner, triggering events, and executing listeners.  
We might want to customize these startup activities performed by the `SpringApplication` class — this is known as **customizing the Spring Boot application**.

```java
ApplicationContext context = SpringApplication.run(Application.class, args);
````

Here we are calling the static `run()` method on the `SpringApplication` class, which bootstraps the Spring Boot application with default activities. This is the quickest way to start the Spring Boot application if we want to run with defaults.

---

## Ways to Customize the Bootstrapping Activities

There are **2 ways** to customize Spring Boot startup behavior:

---

### 1. Through Configuration Approach

In Spring Boot configuration options, we can add Spring Boot–related settings that will be read by the static `run()` method to customize startup activities.

---

### How can we customize the banner through configuration?

#### `application.properties`

```properties
#spring.main.banner-mode=off
#spring.banner.location=application-banner.txt
```

---

### Banner Files

* **banner.txt**
  (must be placed under `src/main/resources` with the file name `banner.txt`)
  → Text inside this file is used to generate the default custom banner.

* **application-banner.txt**
  (or any custom file name you configure)
  → Text inside this file will be used to generate a different banner.



# Example Program for Configuration Approach

---

## Robot.java

```java
@Component
public class Robot {

    @Value("${name}")
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Robot{" +
                "name='" + name + '\'' +
                '}';
    }
}
````

---

## FluentBuliderApiApplication.java

```java
@SpringBootApplication
public class FluentBuliderApiApplication {

    public static void main(String[] args) {
        
        ApplicationContext context = SpringApplication.run(FluentBuliderApiApplication.class, args);

        Robot robot = context.getBean("robot", Robot.class);
        System.out.println(robot);
    }

}
```

---

## application.properties

```properties
name=RobotGenz
spring.main.banner-mode=off  # turn off the banner
spring.banner.location=application-banner.txt  # custom banner file placed in src/main/resources
```

> By placing a custom banner file in the classpath and defining its location in `application.properties`, Spring Boot automatically loads it through the `SpringApplication` class.


# 2. Programmatic Approach

---

## Overview

- If we want to customize the bootstrap activities of a Spring Boot application, we must create a `SpringApplication` object, configure it with required settings, and then call its `run()` method.
- All customizations are **not** available through configuration files. Only a few can be done using `application.properties`.  
  For complete control, we use the **Programmatic Approach**.

---

# Programmatic Approach

To customize the startup:

1. Create an object of `SpringApplication`
2. Populate all required configurations/settings
3. Call the **instance `run()`** method

Spring Boot provides a convenient **fluent builder API** class called `SpringApplicationBuilder` to simplify this process.

---

# Fluent Builder API

## Why do we need builder classes?

Example:

```java
class A {
    int i;
    int j;
    // 20 attributes
}
````

Creating an object traditionally:

```java
A a = new A();
a.setI(10);
a.setJ(20);
// ... set 20 more attributes
```

This makes code lengthy and difficult to maintain.

### Builder Pattern Example

```java
class ABuilder {  // Responsible for creating A with values

    int i;
    int j;
    // 20 attributes

    ABuilder i(int i) {  // fluent setter
        this.i = i;
        return this;
    }

    int i() { // getter
        return this.i;
    }

    ABuilder j(int j) {
        this.j = j;
        return this;
    }

    int j() {
        return this.j;
    }

    public A build() {  // Not a static method
        A a = new A();
        a.setI(this.i);
        a.setJ(this.j);
        return a;
    }
}
```

Usage:

```java
ABuilder abuilder = new ABuilder();

// Traditional way (problematic)
abuilder.setI(10);
abuilder.setJ(20);

// Fluent builder way (recommended)
A a = abuilder.i(10).j(20).build();
```

---

# SpringApplication (Programmatic Customization)

* Instantiate a `SpringApplication` object
* Populate multiple configurations/settings
* But there are many settings, so using setters becomes tedious

### Solution → **SpringApplicationBuilder** (Fluent API)

```java
SpringApplicationBuilder builder = new SpringApplicationBuilder();
builder = builder.sources(Application.class);
builder = builder.bannerMode(Mode.OFF);

SpringApplication application = builder.build();
ApplicationContext context = application.run(args);
// We cannot pass configuration class here again, it is already set using the builder.
```

---

# Single-line Fluent Builder Example

```java
SpringApplication application = 
        new SpringApplicationBuilder()
            .sources(Application.class)
            .bannerMode(Mode.OFF)
            .build();

application.run(args);
```

---

# Note for Spring Boot 2.x

```java
// Custom banner loading (Spring Boot 2.x only)
builder = builder.banner(
        new ResourceBanner(new ClassPathResource("application-banner.txt"))
);
```

Spring Boot **3.x removed `ResourceBanner` intentionally** as part of a larger cleanup and modernization of the startup API.
Here is the **exact reason** and the **background behind the removal**:

---

# ✅ **Why `ResourceBanner` Was Removed in Spring Boot 3.x**

Spring Boot 3 performed a **major refactoring** of its internal codebase due to:

### **1. Migration to Jakarta EE 10**

Spring Boot 3 moved from `javax.*` to `jakarta.*`, which required cleanup of older, unused, or redundant classes.

`ResourceBanner` was considered an **unnecessary wrapper** because:

* It only loaded a banner from a `Resource`.
* The same behavior could be implemented easily using a simple lambda.
* Very few projects used it directly.

---

# **2. Banner API Simplification**

Spring Boot maintainers simplified the banner API to make it:

* smaller
* more maintainable
* less complex

They kept only:

* `Banner`
* `ImageBanner` (special case)
* default auto-loading (`banner.txt`, `spring.banner.location`)

Everything else was removed.

`ResourceBanner` was removed along with other deprecated or redundant utilities.

---

# **3. Encourage the Simpler Standard Approach**

Spring Boot already supports:

### ✔ `banner.txt` in classpath

### ✔ `spring.banner.location=classpath:custom.txt`

Because these already cover 99% of use cases, maintaining a special `ResourceBanner` class had **no real advantage**.

The maintainers prefer:

```properties
spring.banner.location=classpath:application-banner.txt
```

instead of programmatic setting.

---

# **4. Codebase Cleanup for Long-Term Maintenance**

Spring Boot 3 removed multiple internal classes that:

* were rarely used
* added no real value
* required maintenance effort

`ResourceBanner` was part of that pruning.

---

# 📌 Official Explanation (summarized from Spring Boot team discussions)

> The banner loading mechanism was simplified. The `ResourceBanner` type was removed because a banner resource can be easily loaded manually and the preferred banner mechanism is via configuration rather than code.

---

# **5. Encourages Functional-Style Banner Customization**

Instead of:

```java
new ResourceBanner(resource);
```

Spring Boot now prefers:

```java
builder.banner((env, source, out) -> out.print(...));
```

This gives developers **full control** without needing a dedicated class.

---

# ⭐ Summary

| Reason                              | Explanation                                         |
| ----------------------------------- | --------------------------------------------------- |
| API cleanup                         | Remove rarely used classes to keep Boot lightweight |
| Simplification                      | Standard banner mechanism is already enough         |
| Jakarta EE migration                | Many classes refactored / removed                   |
| Maintainability                     | Fewer classes = easier long-term maintenance        |
| Modern functional programming model | Prefer lambdas over small wrapper classes           |

---

# Example Program for Programmatic Approach

---

## Robot.java

```java
@Component
public class Robot {

    @Value("${name}")
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Robot{" +
                "name='" + name + '\'' +
                '}';
    }
}
````

---

## FluentBuliderApiApplication.java

```java
@SpringBootApplication
public class FluentBuliderApiApplication {

    public static void main(String[] args) {

        SpringApplicationBuilder builder = new SpringApplicationBuilder(FluentBuliderApiApplication.class);
        //builder = builder.bannerMode(Banner.Mode.OFF);

        /*
        This is Anonymous Inner class to load the custom banner name
        Banner is Functional Interface so convert into lambda expression

        Banner banner = new Banner() {
            @Override
            public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) {
                try {
                    System.out.println("Source class---" + sourceClass + " " + environment);
                    String ban = new String(new ClassPathResource("application-banner.txt")
                                .getInputStream().readAllBytes());
                    out.println(ban);
                } catch (IOException e) {
                    System.out.println("UNABLE TO LOAD BANNER FILE");
                }
            }
        };
        */

        builder.banner((environment, sourceClass, out) -> {
            try {
                String printBanner = new String(
                    new ClassPathResource("application-banner.txt")
                        .getInputStream().readAllBytes()
                );
                out.println(printBanner);
            } catch (IOException e) {
                System.out.println("UNABLE TO LOAD BANNER");
            }
        });

        // This code is only used for Spring Boot 2.x for custom banner:
        // builder = builder.banner(new ResourceBanner(new ClassPathResource("application-banner.txt")));

        SpringApplication springApplication = builder.build();
        ApplicationContext context = springApplication.run(args);
        // ApplicationContext context = SpringApplication.run(FluentBuliderApiApplication.class, args);

        Robot robot = context.getBean("robot", Robot.class);
        System.out.println(robot);
    }

}
```
````md
# Line-by-Line Explanation of the Code

Below is the explanation of this Spring Boot programmatic banner customization:

```java
builder.banner((environment, sourceClass, out) -> {
    try {
        String printBanner = new String(
            new ClassPathResource("application-banner.txt")
                .getInputStream().readAllBytes()
        );
        out.println(printBanner);
    } catch (IOException e) {
        System.out.println("UNABLE TO LOAD BANNER");
    }
});
````

## 🔍 Line-by-Line Breakdown

### **1. `builder.banner((environment, sourceClass, out) -> {`**

* `builder.banner()`
  Sets a **custom banner** programmatically for the Spring Boot application.
* The parameter is a **lambda expression**, implementing the `Banner` functional interface.
* The lambda receives **three arguments**:

  * `environment` → Gives access to application properties and environment settings.
  * `sourceClass` → The main application class passed to `SpringApplicationBuilder`.
  * `out` → A `PrintStream` used to print the banner to the console.

This lambda replaces the default Spring Boot banner logic.

---

### **2. `try {`**

A **try–catch block** is used because file reading can throw an `IOException`.

---

### **3. `String printBanner = new String(new ClassPathResource("application-banner.txt").getInputStream().readAllBytes());`**

Breakdown:

* `new ClassPathResource("application-banner.txt")`
  Looks for the file `application-banner.txt` inside the **classpath** (`src/main/resources`).

* `.getInputStream()`
  Opens an input stream to read the file contents.

* `.readAllBytes()`
  Reads the entire file into a byte array.

* `new String(...)`
  Converts the byte array into a **String**, storing the banner text in `printBanner`.

This line essentially loads your custom banner text from the resource file.

---

### **4. `out.println(printBanner);`**

* Prints the **custom banner content** to the console.
* `out` is a `PrintStream` provided by Spring Boot for banner printing.

---

### **5. `} catch (IOException e) {`**

If the file is missing or unreadable, the code enters the `catch` block.

---

### **6. `System.out.println("UNABLE TO LOAD BANNER");`**

Instead of crashing the application, it prints an error message gracefully.

---

### **7. `});`**

Closes the lambda expression and the `banner()` method call.

---

# ✔ Summary

This code **replaces the default Spring Boot banner** by manually reading a text file from the classpath and printing it.
It uses:

* Lambda expression implementing `Banner`
* `ClassPathResource` to locate the banner file
* Streams to read the file’s content
* Custom error handling

