# Spring Boot MVC Internal Flow

```java
SpringApplication.run(Config.class, args);
````

1. Creates an empty **Environment** object.
2. Detects and loads the external configurations into the **Environment** object.
3. Prints the Spring **banner**.
4. Detects / identifies the **WebApplicationType** of the application and instantiates the appropriate **IoC container**.

   * **WEB** → `AnnotationConfigServletWebServerApplicationContext`
   * **REACTIVE** → `AnnotationConfigReactiveWebServerApplicationContext`
   * **NONE** → `AnnotationConfigWebApplicationContext`
5. Instantiates and registers the **Spring factories** (auto-configuration classes).
6. Calls / invokes **ApplicationContextInitializers**.
7. Calls **prepareContext**.
8. **refreshContext**
   - Before the refresh happens, the `onRefresh()` method of  
     `AnnotationConfigServletWebServerApplicationContext` is called.

---

### What happens during `onRefresh()` method call?

- Inside the `onRefresh()` method of  
  `AnnotationConfigServletWebServerApplicationContext`, the **embedded servlet container** is created.

---

### How is the Embedded Servlet Container Created?

- Spring Boot supports **3 embedded servlet containers**:
  1. **Tomcat Server**
  2. **Jetty Server**
  3. **Netty Server**

---

### WebServer Abstraction

- Spring Boot introduced an interface called **`WebServer`**.
- Since Spring Boot supports multiple embedded servers, each having its own internal logic,  
  Spring Boot follows an **interface-based design approach**.

---

### WebServer Interface and Implementations

```text
              WebServer (I)
                    |
     ---------------------------------------
     |                 |                   |
TomcatWebServer(C)  JettyWebServer(C)  NettyWebServer(C)
````

---

### WebServerFactory

* Whenever object creation is **complex** or the creation logic of
  `TomcatWebServer`, `JettyWebServer`, or `NettyWebServer` is not known directly,
  Spring Boot uses a **Factory pattern**.

* The factory interface used is **`WebServerFactory`**.

---

### WebServerFactory Implementations

1. `TomcatServletWebServerFactory`
2. `NettyServletWebServerFactory`
3. `JettyServletWebServerFactory`

- Inside the `onRefresh()` method of  
  `AnnotationConfigServletWebServerApplicationContext`, there is an attribute called  
  **`AbstractServletWebServerFactory getWebServerFactory()`**.  

- `AbstractServletWebServerFactory` is an **abstract class** that holds the logic for the  
  three web server implementation classes.

---

- The **IoC Container** identifies the appropriate **WebServer** to be used based on the  
  Spring Boot starter dependency (for example, `spring-boot-starter-tomcat`) available  
  on the **classpath** of the application.

- Based on this detection, the IoC Container instantiates the corresponding  
  **WebServerFactory** implementation, such as `TomcatServletWebServerFactory`.

---

- After instantiating the `TomcatServletWebServerFactory` object, it reads the required  
  configuration from the **Environment** object of the IoC Container and then creates  
  the corresponding implementation of the `WebServer` interface  
  (for example, `TomcatWebServer`).

---

- After that, the IoC Container invokes the `webServer.start()` method.  
  At this point, the **embedded Tomcat server** is started.

---

## Spring Boot Internal Classes Hierarchy

```text
WebServerFactory
   └─ AbstractServletWebServerFactory
        └─ TomcatServletWebServerFactory
             └─ getWebServer()
````

```text
WebServer
 ├─ TomcatWebServer
 │    └─ Tomcat
 ├─ JettyWebServer
 └─ NettyWebServer
```

- Once the **embedded Tomcat server** is started, the next step is to register the  
  **DispatcherServlet** with the **ServletContext**.  
  Below are the steps involved:

---

- Spring Boot provides an interface called **`ServletContextInitializer`**.  
  - This acts like **programmatic registration** of the DispatcherServlet with the  
    `ServletContext`.

---

- During the startup of the **embedded servlet container**, it invokes the  
  **`ServletContextInitializer`**.

  - The `ServletContextInitializer` searches the **bean definitions** in the  
    **IoC Container** that are of type:
    - `ServletRegistrationBean`
    - `FilterRegistrationBean`

  - These beans are then **added / registered** with the `ServletContext` of the  
    embedded servlet container.

---

- `ServletRegistrationBean` and `FilterRegistrationBean` hold the configuration  
  information about the **Servlets** and **Filters** that should be registered  
  with the **Servlet container**.

---

- For the **DispatcherServlet**, Spring Boot provides:
  - `DispatcherServletRegistrationBean`

- The `DispatcherServletAutoConfiguration` class configures the  
  `DispatcherServletRegistrationBean` as a **bean definition** by reading values  
  from the **Environment** properties.
  
## Summary of What Happens During the `onRefresh()` Method of the IoC Container

1. The IoC Container identifies the appropriate **embedded servlet container** within the application and accesses the corresponding **WebServerFactory** interface implementation available in the IoC Container.

2. If the embedded server is **Tomcat**, the IoC Container obtains the  
   `TomcatServletWebServerFactory` and invokes the `getWebServer()` method.

3. The `getWebServer()` method instantiates the appropriate implementation  
   (`TomcatWebServer`) of the `WebServer` interface by populating the required  
   configuration and then starts the embedded servlet container.

---

### Servlet Registration During Embedded Server Startup

- During the startup of the **embedded servlet container**, it invokes the  
  **`ServletContextInitializer`**.

  - The `ServletContextInitializer` searches the **bean definitions** in the  
    IoC Container that are of type:
    - `ServletRegistrationBean`
    - `FilterRegistrationBean`

  - These bean definitions are then **added / registered** with the  
    `ServletContext` of the embedded servlet container.

---

- `ServletRegistrationBean` and `FilterRegistrationBean` hold the configuration  
  details of the **Servlets** and **Filters** that need to be registered with the  
  **Servlet container**.

---

- For the **DispatcherServlet**, Spring Boot provides:
  - `DispatcherServletRegistrationBean`

- The `DispatcherServletAutoConfiguration` class configures the  
  `DispatcherServletRegistrationBean` as a **bean definition** by reading values  
  from the **Environment** properties.

---

9. Invokes **CommandLineRunners** and **ApplicationRunners**.  
10. Publishes the **ApplicationReadyEvent**.
```
