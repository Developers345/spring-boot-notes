# Project Structure
------------------

```

rapido
├─ src
│  ├─ main
│  │  ├─ java
│  │  └─ resources
│  └─ webapp
│     └─ WEB-INF
│        ├─ jsp
│        │  └─ bikes.jsp
│        └─ web.xml
└─ target

````

---

# Controller Code
-----------------

```java
@Controller
class ListBikesController {

    @Autowired
    private ManageBikeService managedBikeService;

    /*
     Here problem is whenever DispatcherServlet calls the showBikes method,
     DispatcherServlet passes the HttpServletRequest object.
     So controller class works with servlet-specific components only.
     Normal Java classes cannot call it.
    
    public String showBikes(HttpServletRequest req) {
        bikes = managedBikeService.getBikes();
        req.setAttribute("bikes", bikes);
    }
    */

    /*
     To overcome the above problem, showBikes() can accept a Java Map object
     as a parameter. DispatcherServlet instantiates an empty map object,
     then controller puts the response into that map.
     Other classes in the application can call the controller.
    
    public String showBikes(Map<String, Object> map) {
        bikes = managedBikeService.getBikes();
        map.put("bikes", bikes);
    }
    */

    /*
     The above Map generic never changes, so Spring MVC provides
     Model or ModelMap. DispatcherServlet instantiates an empty
     Model/ModelMap object, then controller puts the response into it
     and returns the logical view name.
     */
    @RequestMapping("/bikes.htm")
    public String showBikes(ModelMap modelMap) {
        bikes = managedBikeService.getBikes();
        modelMap.addAttribute("bikes", bikes);
        return "bikes";
    }
}
````

---

# Spring MVC Request Flow

---

* Whenever the end user sends the request (`/bikes.htm`), the servlet container receives the request and delegates it to **DispatcherServlet**.

* Once the request reaches the **DispatcherServlet**, it applies common processing logic for all incoming requests.

* After applying common processing logic, **DispatcherServlet** calls the **HandlerMapping** implementation by passing the incoming URL (`/bikes.htm`).

* The **HandlerMapping** implementation class used is **RequestMappingHandlerMapping**.

* **RequestMappingHandlerMapping** goes to the IOC container, finds `@Controller` annotated beans, checks all controller methods annotated with `@RequestMapping("/bikes.htm")`, and returns the matched controller method to the **DispatcherServlet**.

* Once the controller method is identified, **DispatcherServlet** takes help from **RequestMappingHandlerAdapter** (internal helper class).

* **RequestMappingHandlerAdapter** is responsible for invoking controller methods by passing appropriate parameters (Model, ModelMap, Request, Response, etc.).

* **DispatcherServlet** creates the IOC container using **WebApplicationContext**, so **RequestMappingHandlerAdapter** is created and placed in the IOC container as a bean definition.

```
HandlerMapping (Interface)
 ├─ RequestMappingHandlerMapping (Implementation)
 └─ RequestMappingHandlerAdapter (Implementation)
```

* Controller adds the response data to the empty **ModelMap** object and returns the logical view name to **DispatcherServlet**.

---

# View Resolution Flow

---

* **DispatcherServlet** calls the **ViewResolver** by passing the logical view name.

* **ViewResolver** is an interface, and its commonly used implementation is **InternalResourceViewResolver**.

* **InternalResourceViewResolver** is used to create the **JstlView** object, which is responsible for rendering JSP pages.

* Once the **View** object is received, **DispatcherServlet** calls the `render()` method on the view object.

---

# ViewResolver Details

---

**ViewResolver (Interface)**
Responsible for instantiating the View class object and returning it to **DispatcherServlet** based on the logical view name.

---

## InternalResourceViewResolver

---

* **InternalResourceViewResolver** is an implementation of the **ViewResolver** interface.

* It performs the following steps:

### 1. Logical View Name Resolution

* Takes the logical view name passed by **DispatcherServlet**.
* Prefixes and suffixes the configured values.

**Example:**

* Logical View Name: `bikes`
* Final JSP Path:

  ```
  /WEB-INF/jsp/bikes.jsp
  ```

### 2. View Object Creation

* Instantiates a **JstlView** object by default.

* Populates the constructed JSP path as the URL in the **JstlView** object.

* Returns the **View** object to **DispatcherServlet**.

* **InternalResourceViewResolver** is suitable for rendering JSP pages because it works with **JstlView** by default.

---

```
ViewResolver (Interface)
 └─ InternalResourceViewResolver (Implementation)
     ├─ JstlView (JSP only)
     ├─ prefix = "/WEB-INF/jsp/"
     └─ suffix = ".jsp"
```

---

# JSTLView

---

**JSTL (Java Standard Tag Library)**

* **JstlView** displays the JSP page when its `render()` method is called.

```java
render(request, response) {
    request.getRequestDispatcher("/WEB-INF/jsp/bikes.jsp")
           .forward(request, response);
}
```
# Partical program for Spring MVC configuration approach refer below git repo
https://github.com/Developers345/practical-spring-mvc-configuration-approach-example.git
