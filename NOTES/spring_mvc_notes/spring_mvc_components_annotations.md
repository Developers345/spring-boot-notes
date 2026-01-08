## How to Configure DispatcherServlet Programmatically  
### (Without using `web.xml`)

---

- Servlet Container provides an interface called **`ServletContainerInitializer`**, which has a method  
  **`onStartup(ServletContext context)`**.  
  This method receives the **ServletContext** object as a parameter.

- The Servlet Container will call the **`ServletContainerInitializer`** implementation class and its  
  **`onStartup()`** method by passing the **ServletContext** object **after completion of ServletContext instantiation**.

- The Servlet Container will **not call the `ServletContainerInitializer` implementation directly**  
  because it does not know which classes need to be registered with the `ServletContext`.  
  To inform the container, we must create a file under:

```

META-INF/services/jakarta.servlet.ServletContainerInitializer

```

The Servlet Container reads this file and registers the listed initializer classes with the `ServletContext`.

- While deploying a web application, the Servlet Container creates an object called **ServletContext**.

- One **ServletContext** object is created per **one web application**.
- All application configuration such as:
  - servlets
  - listeners
  - filters
  - session configuration
  - security configuration  
  is stored inside the ServletContext.

- Configuration that was traditionally written in `web.xml` (or fully using annotations) is placed  
into the **ServletContext** during or after deployment.

- Once the **ServletContext** object is created successfully, the **web application deployment is considered successful**.


# The Flow of Configured DispatcherServlet and ContextLoaderListener

---

## [Servlet Container]

During the deployment of the application:

- After creating the **ServletContext** for the web application  
- **Before** instantiating any web application components  

The **Servlet Container** calls classes that implement the  
`ServletContainerInitializer` interface by invoking:

```

onStartup(ServletContext context)

```

---

## Flow Diagram

```

[Servlet Container]
|
v
ServletContext
|
v
-

SpringServletContainerInitializer
(Spring MVC internally provided class)
|
v
onStartup(ServletContext context)
|
v
Looks for classes implementing
WebApplicationInitializer interface
|
v
Calls onStartup(ServletContext context)

```

---

## Key Points

- `ServletContainerInitializer` is invoked by the **Servlet Container**
- `SpringServletContainerInitializer` is provided internally by **Spring MVC**
- It scans for implementations of `WebApplicationInitializer`
- Each `WebApplicationInitializer` gets a callback through:
```

onStartup(ServletContext context)

```
- This is where **DispatcherServlet** and **ContextLoaderListener** are typically configured programmatically


## DispatcherServlet and ContextLoaderListener – IOC Container Creation

---

### web.xml Configuration Approach

- Whenever we use the **web.xml configuration approach**, the IOC containers created for **DispatcherServlet** and **ContextLoaderListener** are **fixed**.

- When we define `dispatcher-servlet.xml` and use:

```xml
<mvc:annotation-driven/>
````

* Then **`AnnotationConfigWebApplicationContext`** IOC container will be created.

* If you **do NOT** mention the `<mvc:annotation-driven/>` tag in `dispatcher-servlet.xml`, then:

* **`XmlWebApplicationContext`** IOC container will be created.

---

### Programmatic Configuration Approach

* Unlike the **web.xml approach**, in **programmatic configuration** the developer has full control.

* The developer can explicitly decide **which type of IOC container** should be used by:

  * **DispatcherServlet**
  * **ContextLoaderListener**

* In this approach:

  * IOC containers are created using **Java configuration classes**
  * These containers are passed as input to:

    * `DispatcherServlet`
    * `ContextLoaderListener`

---

### Summary

* **web.xml approach** → IOC container type is predefined and limited
* **Programmatic approach** → IOC container type is flexible and developer-controlled


## Sample Code to Configure DispatcherServlet Programmatically

---

### Overview

- Spring Framework provides the **`WebApplicationInitializer`** interface.
- `WebApplicationInitializer` is internally integrated with  
  **`ServletContainerInitializer`**.
- Developers **do NOT** need to directly implement  
  `ServletContainerInitializer`.

---

### Programmatic Configuration Example

```java
import jakarta.servlet.ServletContext;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRegistration;

import org.springframework.web.WebApplicationInitializer;
import org.springframework.web.context.ContextLoaderListener;
import org.springframework.web.context.support.AnnotationConfigWebApplicationContext;
import org.springframework.web.servlet.DispatcherServlet;

public class ParcelTrackingWebApplicationInitializer
        implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext context) throws ServletException {

        // Root Application Context
        AnnotationConfigWebApplicationContext rootApplicationContext =
                new AnnotationConfigWebApplicationContext();
        rootApplicationContext.register(RootConfig.class);

        ContextLoaderListener contextLoaderListener =
                new ContextLoaderListener(rootApplicationContext);
        context.addListener(contextLoaderListener);

        // Web (MVC) Application Context
        AnnotationConfigWebApplicationContext webApplicationContext =
                new AnnotationConfigWebApplicationContext();
        webApplicationContext.register(MvcConfig.class);

        // DispatcherServlet
        DispatcherServlet dispatcherServlet =
                new DispatcherServlet(webApplicationContext);

        ServletRegistration.Dynamic dynamic =
                context.addServlet("dispatcher", dispatcherServlet);

        dynamic.setLoadOnStartup(1);
        dynamic.addMapping("*.htm");
    }
}
````

---

### Key Points

* `WebApplicationInitializer` enables **programmatic servlet configuration**
* `ContextLoaderListener` is responsible for creating the **root IOC container**
* `DispatcherServlet` uses a **separate web IOC container**
* Java configuration classes (`RootConfig`, `MvcConfig`) are used instead of `web.xml`
* This approach fully replaces **web.xml** configuration

  
## How to Configure Spring MVC Components  
### (HandlerMappings, ViewResolvers) to IOC Container

---

### Spring MVC Components Overview

- There are many **Spring MVC components** such as:
  - `HandlerMapping`
  - `InternalResourceViewResolver`
  - `LocaleResolver`
  - `ThemeResolver`
  - etc.

- Most of these components are **required** to run a Spring MVC application.

- Among them:
  - Some components are **used internally** by Spring MVC
  - Some components **must be configured by the developer** based on application needs

---

### Problem Faced by Developers

- To run a Spring MVC application, **hundreds of framework classes** are involved.
- If developers had to configure all these classes manually:
  - They would spend more time on **framework configuration**
  - Instead of focusing on **business logic**

- This would make application development **very complex**.

---

### Solution: `WebApplicationContext`

- To overcome this problem, Spring MVC introduced an interface called:

```

WebApplicationContext

```

- **Main Purpose**:
  - To create and configure **default Spring MVC objects**
  - Required to run a Spring MVC application

---

### Types of WebApplicationContext

```

WebApplicationContext (Interface)
|
-

|                                              |
XmlWebApplicationContext        AnnotationConfigWebApplicationContext

```

- `XmlWebApplicationContext` → XML-based configuration
- `AnnotationConfigWebApplicationContext` → Java/Annotation-based configuration

---

### Custom Configuration Using WebMVCConfigurerAdapter

- Even though `WebApplicationContext` creates most default MVC objects,
  some components **require external (developer-provided) configuration**.

- For this purpose, Spring MVC provides:

```

WebMVCConfigurerAdapter

```

- Developers only need to provide **configuration details**, not object creation logic.

- `WebMVCConfigurerAdapter` exposes **registries (builders)** such as:

```

List<HandlerMapping>
List<ViewResolver>
List<MessageConverter>

```

- Spring MVC automatically:
  - Applies developer configurations
  - Wires them with existing framework components

---

### Note

- `WebMVCConfigurerAdapter` **configures Spring MVC components**
- These components are **already defined as beans** in the IOC container
- Developers only customize behavior, **not bean creation**


## Java Configuration Approach for Spring MVC Components

---

### Why Java Configuration?

- To **avoid configuring Spring MVC components** in Spring bean XML configuration files,
  Spring Framework introduced the **Java configuration approach**.

---

### Manual Configuration Example

```java
@Configuration
class MvcConfig {

    @Bean
    public HandlerMapping handlerMapper() {
        return new RequestMappingHandlerMapping();
    }
}
````

* In the above configuration:

  * The developer explicitly defines a `HandlerMapping` bean
  * This approach still requires manual bean registration

---

### Using `@EnableWebMvc`

* Spring MVC provides the **`@EnableWebMvc`** annotation to simplify configuration.
* When `@EnableWebMvc` is used:

  * `RequestMappingHandlerMapping`
  * `RequestMappingHandlerAdapter`

  are **automatically registered** in the IOC container.

#### Modified Configuration

```java
@Configuration
@EnableWebMvc
class MvcConfig {}
```

---

### Sample Controller

```java
@Controller
class ParcelController {

    @RequestMapping("/search-products.htm")
    public String showSearchPage() {
        return "search-products";
    }
}
```

* This controller method:

  * Only returns a **logical view name**
  * Contains **no business logic**

---

### Using `ParameterizableViewController`

* For controllers that **only return a view name**, Spring MVC provides:

```
ParameterizableViewController
```

#### Example Configuration

```java
@Configuration
@EnableWebMvc
class MvcConfig {

    @Bean
    public Controller showSearchProducts() {
        ParameterizableViewController viewController =
                new ParameterizableViewController();
        viewController.setViewName("search-products");
        return viewController;
    }
}
```

---

### Optimizing with `WebMVCConfigurerAdapter`

* To further simplify and optimize configuration,
  Spring MVC introduced the **`WebMVCConfigurerAdapter`** class.

#### Optimized Configuration

```java
@Configuration
@EnableWebMvc
class MvcConfig extends WebMVCConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/search-products.htm")
                .setViewName("search-products");
    }

    // ViewResolvers configuration
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp()
                .prefix("/WEB-INF/jsp/")
                .suffix(".jsp");
    }
}
```

---

### Developer Responsibility Simplified

* Developers now **only write controller classes**
* All remaining Spring MVC infrastructure components are:

  * Automatically created
  * Automatically configured
    by the Spring MVC module

---

### Sample Business Controller

```java
class ParcelController {

    @RequestMapping("/search-parcel.htm")
    public String showSearchParcelPage() {
        return "search-parcel";
    }

    @RequestMapping("/doSearchParcel.htm")
    public String searchParcel(
            @ModelAttribute ParcelCriteria parcelCriteria,
            ModelMap modelMap) {
        return "search-results";
    }
}
```

---

### `@ModelAttribute` Explanation

* `@ModelAttribute` indicates:

  * Data submitted from a JSP form
  * Sent in the **request body**
  * In **`application/x-www-form-urlencoded`** format

* Spring MVC:

  * Extracts request body data
  * Binds it to the object annotated with `@ModelAttribute`
  * Makes the object available to the controller method



