# ApplicationEvents and Listeners

Event Handling → Asynchronous, loosely coupled, disconnected application.

## Event-Driven Programming – Four Actors

1. **Source**  
   The originator of the event. When the source wants to perform some action, it triggers an event.

2. **Event**  
   An object encapsulated with data and source information that triggered the event.

3. **Listener**  
   Listens for an event from the source. Upon receiving the event, it identifies the appropriate handler method that can process the event and dispatches it.

4. **Handlers**  
   Contain event-handling methods that receive the event as input and perform the required operation.

---

# Spring Boot Application Events

- In Spring Boot, the `SpringApplication` class is used to start the application.  
- `SpringApplication` performs various activities while bootstrapping the application.

During the bootstrapping process, SpringApplication triggers multiple events which indicate different stages of startup. These events allow developers to perform additional operations.

**(or)**

SpringApplication triggers events at different bootstrapping stages. Developers can register handlers to capture and process these events using Spring Boot’s event-driven programming support.

---

## SpringApplication Bootstrapping Stages

1. Creates the empty Environment object  
2. Detects and loads external configuration into the Environment  
3. Prints the banner  
4. Identifies the type of application and instantiates the appropriate IOC container  
5. Instantiates and registers *Spring Factories* classes  
6. Executes `ApplicationContextInitializer` (IOC container initializer)  
7. `prepareContext`  
8. `refreshContext`  
9. Executes `CommandLineRunner` and `ApplicationRunner`, starts the application, and returns the IOC container  

---

- The **source** is the SpringApplication class triggering predefined events.  
- To process the events, we write a Listener/Handler class.  
- We then **register the listener** with SpringApplication because the IOC container does not exist yet.

---

# Four Actors in Spring Boot Events

| Actor       | Description |
|-------------|-------------|
| **Source**  | `SpringApplication` class |
| **Event**   | Predefined events indicating bootstrapping stages |
| **Handler** | Developer-written handler class |
| **Listener**| SpringApplication acts as listener (because IOC not created yet) |

---

# Registering a Handler Class

```java
class MyHandler implements ApplicationListener<ApplicationStartedEvent> {

    public void onApplicationEvent(ApplicationStartedEvent event) {
        // handle event
    }
}
````

* Some events occur **before** IOC container creation.
* So the container cannot call the listener — SpringApplication itself must invoke it.
* Therefore, we register listeners using **SpringApplicationBuilder** (Fluent Builder API).

---

# Predefined Events in Spring Boot

## 1. ApplicationStartingEvent

* Triggered when `SpringApplication` is created with listeners and initializers.
* No other processing has yet started.
* Indicates SpringApplication is *about to start*.

```java
SpringApplication app = new SpringApplicationBuilder(Application.class)
        .listener(listener)
        .build();
app.run(args);
```

---

## 2. ApplicationEnvironmentPreparedEvent

* Empty Environment object is created.
* The application type (web, webflux, default) is identified.
* IOC container **not yet instantiated**.

---

## 3. ApplicationPreparedEvent

* Environment is ready.
* IOC container is instantiated.
* Bean definitions are loaded but **beans are not yet instantiated**.

---

## 4. ApplicationStartedEvent

* IOC container has instantiated all beans.
* Control is not yet returned to the application.

---

## 5. ApplicationReadyEvent

* CommandLineRunner and ApplicationRunner have executed.
* Application is fully ready for use.

---

## 6. ApplicationFailedEvent

* Triggered when application startup fails due to any exception.

---

# Example Program for Spring Boot Event and Listeners

## BLApplicationStartingEvent.java
```java
public class BLApplicationStartingEvent implements ApplicationListener<ApplicationStartingEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartingEvent event) {
        System.out.println("Application Starting Event:" + " " + event.getClass().getName());
    }
}
````

---

## BLApplicationEnviornmentPreparedEvent.java

```java
public class BLApplicationEnviornmentPreparedEvent implements ApplicationListener<ApplicationEnvironmentPreparedEvent> {
    @Override
    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
        System.out.println("Application Environment Prepared Event:" + " " + event.getEnvironment().getClass().getName());
    }
}
```

---

## BLApplicationPreparedEvent.java

```java
public class BLApplicationPreparedEvent implements ApplicationListener<ApplicationPreparedEvent> {
    @Override
    public void onApplicationEvent(ApplicationPreparedEvent event) {
        System.out.println("Application Prepared Event:" + " " + event.getApplicationContext().getClass().getName());
    }
}
```

---

## BLApplicationStartedEvent.java

```java
public class BLApplicationStartedEvent implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("Application Started Event");
        Toy toy = event.getApplicationContext().getBean("toy", Toy.class);
    }
}
```

---

## BLApplicationReadyEvent.java

```java
public class BLApplicationReadyEvent implements ApplicationListener<ApplicationReadyEvent> {
    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        System.out.println("Application ready to use:" + " " + event.getClass().getName());
        Toy toy = event.getApplicationContext().getBean("toy", Toy.class);
        toy.play();
    }
}
```

---

## Toy.java

```java
@Component
public class Toy {

    public Toy() {
        System.out.println("toy is initializing..");
    }

    public void play() {
        System.out.println("Playing.....");
    }
}
```

---

## EventListenersApplication.java (Test class)

```java
@SpringBootApplication
public class EventListenersApplication {

    public static void main(String[] args) {

        SpringApplication springApplication = new SpringApplicationBuilder(EventListenersApplication.class)
                .listeners(new BLApplicationStartingEvent(),
                        new BLApplicationEnviornmentPreparedEvent(),
                        new BLApplicationPreparedEvent(),
                        new BLApplicationStartedEvent(),
                        new BLApplicationReadyEvent(),
                        new GlobalEvent())
                .build();

        springApplication.run(args);
        System.out.println("Application is running...");
    }
}
```

---

# Output:

```
Application Starting Event: org.springframework.boot.context.event.ApplicationStartingEvent
Application Environment Prepared Event: org.springframework.boot.ApplicationEnvironment

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.4.0)

2025-12-06T07:29:49.491+05:30  INFO 9452 --- [           main] com.bl.EventListenersApplication         : Starting EventListenersApplication using Java 17.0.11 with PID 9452 (D:\Core java\sspringboot\boot-event-listeners\target\classes started by Dell in D:\Core java\sspringboot\boot-event-listeners)
2025-12-06T07:29:49.493+05:30  INFO 9452 --- [           main] com.bl.EventListenersApplication         : No active profile set, falling back to 1 default profile: "default"
Application Prepared Event: org.springframework.context.annotation.AnnotationConfigApplicationContext
toy is initializing..
2025-12-06T07:29:49.974+05:30  INFO 9452 --- [           main] com.bl.EventListenersApplication         : Started EventListenersApplication in 0.839 seconds (process running for 1.077)
Application Started Event
Application ready to use: org.springframework.boot.context.event.ApplicationReadyEvent
Playing.....
Application is running...

Process finished with exit code 0
```

| Stage | Event Name                              | What Happens Here                              | IOC Container?                        |
| ----- | --------------------------------------- | ---------------------------------------------- | ------------------------------------- |
| 1     | **ApplicationStartingEvent**            | SpringApplication object created               | ❌ Not created                         |
| 2     | **ApplicationEnvironmentPreparedEvent** | Environment prepared, config loaded            | ❌ Not created                         |
| 3     | **ApplicationPreparedEvent**            | IOC container created, bean definitions loaded | ✔️ Created but beans NOT instantiated |
| 4     | **ApplicationStartedEvent**             | Beans instantiated, context refreshed          | ✔️ Fully ready                        |
| 5     | **ApplicationReadyEvent**               | Runners executed, app ready                    | ✔️ Running                            |
| 6     | **ApplicationFailedEvent**              | Startup failure occurred                       | ❌/✔️ Depends on failure               |
