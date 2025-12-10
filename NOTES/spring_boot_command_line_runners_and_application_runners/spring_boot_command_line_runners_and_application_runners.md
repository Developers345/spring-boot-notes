# CommandLineRunners and ApplicationRunners

* **CommandLineRunners** or **ApplicationRunners** are used to perform startup activities in a Spring Boot application.
                        (or)

* These are used for performing initialization activities during the bootstrapping of a Spring Boot application, **after the IoC container has been fully instantiated** and **before the application begins execution**.

---
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

* The `SpringApplication.run()` method calls the **CommandLineRunners** and **ApplicationRunners** by passing `args` as input **after `refreshContext` has completed successfully**.
* Once the CommandLineRunners and ApplicationRunners finish execution, the **IoC container is returned to the application**.

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

### Servlet Applications(Pictorial Representation)
<img width="854" height="599" alt="Screenshot 2025-12-10 071025" src="https://github.com/user-attachments/assets/8e0d2c24-aedb-4868-942f-6974a619389d" />

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


### SpringWeb Mvc Application(Pictorial Representation)
<img width="1126" height="559" alt="Screenshot 2025-12-10 071048" src="https://github.com/user-attachments/assets/30e2826b-3858-47e4-bc1c-aac66882b193" />

---
## Use Case

* If you want to initialize the cache **after the IoC container has been created** and **after `refreshContext` has completed**, then you can load the values into the cache **before the IoC container is returned to the developer**.
* The best place to perform this activity is **CommandLineRunners / ApplicationRunners**.

---

# Example Program for Spring Boot Application CommandLineRunners and ApplicationRunners

---

## Address.java

```java
@Component
@ConfigurationProperties(prefix = "address")
public class Address {

    protected String addressLine1;
    protected String AddressLine2;
    protected String city;
    protected String state;
    protected String country;

    public String getAddressLine1() {
        return addressLine1;
    }

    public void setAddressLine1(String addressLine1) {
        this.addressLine1 = addressLine1;
    }

    public String getAddressLine2() {
        return AddressLine2;
    }

    public void setAddressLine2(String addressLine2) {
        AddressLine2 = addressLine2;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }

    @Override
    public String toString() {
        return "Address{" +
                "addressLine1='" + addressLine1 + '\'' +
                ", AddressLine2='" + AddressLine2 + '\'' +
                ", city='" + city + '\'' +
                ", state='" + state + '\'' +
                ", country='" + country + '\'' +
                '}';
    }
}
```

---

## BootRunnerApplication.java

```java
@SpringBootApplication
@PropertySource("classpath:boot-runner.properties")
public class BootRunnerApplication {

    public static void main(String[] args) {

        ApplicationContext context = SpringApplication.run(BootRunnerApplication.class, args);
        try {
            Address address = context.getBean(Address.class);
            System.out.println(address);
        } finally {
            System.exit(SpringApplication.exit(context));
        }
    }

    @Component
    @Order(1)
    private final class ApplicationContextProfilerCommandLineRunnner
            implements CommandLineRunner, ApplicationContextAware {

        private ApplicationContext applicationContext;
        String[] beanNames = null;

        @Override
        public void run(String... args) throws Exception {

            System.out.println("---------------args--------------------------");
            Stream.of(args).forEach(System.out::println);
            System.out.println("---------------args--------------------------");

            System.out.println("************* Bean Names inside IOC Container*********************");
            beanNames = applicationContext.getBeanDefinitionNames();
            Stream.of(beanNames).forEach(System.out::println);
            System.out.println("************* Bean Names inside IOC Container*********************");
        }

        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            this.applicationContext = applicationContext;
        }
    }

    @Component
    @Order(2)
    private final class ExploreArugmentsApplicationRunner
            implements ApplicationRunner, ApplicationContextAware {

        private ApplicationContext applicationContext;
        private String[] beanNames = null;
        List<String> nonOptionArgs = null;
        Set<String> optionNames = null;

        @Override
        public void run(ApplicationArguments args) throws Exception {

            System.out.println("******************* non-option-arguments ******************");
            nonOptionArgs = args.getNonOptionArgs();
            nonOptionArgs.stream().forEach(System.out::println);
            System.out.println("******************* non-option-arguments ******************");

            System.out.println("--------------------- option-arguments --------------------");
            optionNames = args.getOptionNames();
            optionNames.stream().forEach(optionName -> {
                System.out.println(optionName + " : " + args.getOptionValues(optionName).get(0));
            });
            System.out.println("--------------------- option-arguments --------------------");

            System.out.println(">>>>>>>>>>>>>>>>>>> Bean Definitions in IOC Container Application Runner >>>>>>>>>>>>>>>>>>>");
            beanNames = applicationContext.getBeanDefinitionNames();
            Stream.of(beanNames).forEach(System.out::println);
            System.out.println(">>>>>>>>>>>>>>>>>>> Bean Definitions in IOC Container Application Runner >>>>>>>>>>>>>>>>>>>");
        }

        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            this.applicationContext = applicationContext;
        }
    }
}
```


