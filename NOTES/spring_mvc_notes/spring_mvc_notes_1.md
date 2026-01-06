# Internal of Request Processing in Spring MVC

- Once the request comes to **DispatcherServlet**, then it will be passed to the controller for processing,  
  further processing is done with the help of **HandlerMapping**.

- To receive each and every request, first we have to configure **DispatcherServlet** in `web.xml`
  with a standard URL pattern.

- For a servlet-name, we can use any name, but it is recommended to use `dispatcher`, and the
  `servlet-class` should be a fully qualified name.

- Once **DispatcherServlet** has been configured in `web.xml`, now we have to look for other
  components which help Dispatcher to fulfill the request processing.

- Other components are **HandlerMapping**, **ViewResolver**, and **Controller**.  
  These components are written by the programmer because the programmer knows:
  - which URL pattern maps to which controller  
  - which view should be rendered when the controller returns a logical view name to the DispatcherServlet

- These things are known to the programmer, but we have to tell the **DispatcherServlet** that
  these components help it to fulfill request processing.  
  To tell the DispatcherServlet, we have to write a Spring bean configuration file which contains
  all the additional components as beans.

- But how will DispatcherServlet read this Spring bean configuration file?  
  For that, we have to follow some standard while writing the bean file.

- The Spring bean file name should be **`dispatcher-servlet.xml`**  
  (`<servlet-name>-servlet.xml`).  
  Only then will DispatcherServlet be able to read that file.

- While servlet loading, in the `init()` method, it will read the `dispatcher-servlet.xml` file and
  place it as an IOC container in logical memory.  
  This means `load-on-startup` — all beans will be created and placed into the logical memory of
  the IOC container.

- Now, **DispatcherServlet** can easily talk to other components to accomplish request processing
  because all are part of the same IOC container.

- Apart from controller and viewResolver, other components also exist, such as **Service**, **DAO**, etc.  
  There may be hundreds of controllers, services, and DAOs.  
  Placing all of them into one Spring bean configuration file is very difficult.

- Some beans are presentation-specific, while others belong to business and persistence layers.
  Mixing them makes it difficult to identify components, reduces readability, and mixes
  presentation logic with business and persistence logic.

- If we want to change the presentation tier, all business and persistence logic gets affected
  because everything is part of one Spring bean configuration file.

- To keep Business and Persistence tiers safe, we should not mix them with the presentation tier.

- Write one more configuration file that contains only Business tier and Persistence tier beans.
  But writing the configuration file alone is not enough — we must tell DispatcherServlet (or someone)
  to load this file into the IOC container.

- Now check dependencies:
  - Controller talks to Service
  - Service talks to DAO

- If Service exists, we can inject Service into Controller.
- If DAO exists, we can inject DAO into Service.

- This means before loading DispatcherServlet, the other bean file must be loaded first.
  Only then can we inject Service into Controller and DAO into Service.

- The name of this new bean file is **`application-context.xml`**.

- The component that gets loaded first when the `ServletContext` object is created is a **ContextListener**.

- Listeners are classes that get loaded earlier than any other components.
  To load `application-context.xml`, Spring provides a listener called **ContextLoaderListener**.

- We must configure **ContextLoaderListener** in `web.xml` and specify `application-context.xml`
  so that ContextLoaderListener can load it.


# What Is the Reason to Create Two IOC Containers and Inject Them with Each Other?

- First of all, Spring is not forcing the programmer to create two IOC containers.

- **`dispatcher-servlet.xml`**  
  - This configuration file is specific to **Spring MVC**.  
  - It mainly contains presentation-layer components such as:
    - Controllers  
    - HandlerMappings  
    - ViewResolvers  

- **`applicationContext.xml`**  
  - This configuration file is meant for **Business layer** and **Persistence layer** components.  
  - It contains beans such as:
    - Services  
    - DAOs  
    - Repositories  

- The main reason for separating these two configurations is **loose coupling and better architecture**.

- In the future, if we decide **not to use Spring MVC**, we can easily remove or replace the
  Spring MVC layer without disturbing the existing **business tier logic**.

- Since business and persistence logic are kept independent of the web layer,
  we can replace any web component (Spring MVC, another MVC framework, or even a REST API)
  without affecting the core business functionality.

- This separation improves:
  - Maintainability  
  - Reusability  
  - Readability  
  - Flexibility of the application architecture


# How Internally Both IOC Containers Get Created and Interact with Each Other?

- First, **ContextLoaderListener** creates an IOC container by reading
  **`application-context.xml`**, which we configure in `web.xml`.

- It reads the Spring bean configuration file and creates an IOC container by
  placing all the defined beans into the IOC container.

- Once the IOC container is created, the reference of that IOC container is
  placed into the **ServletContext**.

- When **DispatcherServlet** starts creating its own IOC container, before
  creating it, the DispatcherServlet checks whether any **parent IOC container
  reference** is available in the ServletContext or not.

- If a parent IOC container reference is available, DispatcherServlet takes that
  reference and uses it as the **parent context** while creating its own IOC
  container.

- As a result, the DispatcherServlet creates a **child IOC container** that can
  interact with the parent IOC container.

- The interaction follows a **parent–child context hierarchy**, where:
  - The child context (DispatcherServlet IOC) can access beans from the parent context
  - The parent context cannot access beans from the child context

- Conceptual representation (not exact internal code):

```java
ApplicationContext parent =
        new ClassPathXmlApplicationContext("applicationContext.xml");

ApplicationContext child =
        new WebApplicationContext("dispatcher-servlet.xml", parent);
````

* This mechanism allows:

  * Controllers (child context) to use Services and DAOs (parent context)
  * Clear separation between presentation layer and business/persistence layers
  * Easy replacement of the web layer without affecting business logic

# Pictrial Representation 
<img width="1899" height="715" alt="Screenshot 2026-01-06 124048" src="https://github.com/user-attachments/assets/a552c421-5645-4674-a5aa-74b1aab007b1" />

