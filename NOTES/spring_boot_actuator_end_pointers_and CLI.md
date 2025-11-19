# Actuator Endpoints
---------------------

- Developed Code → Production Deployable → **No**  
  Additional **metrics and monitoring endpoints** must be added to the code to make it ready for production deployment.

- All such monitoring and operational endpoints come as part of **Spring Boot Actuator**.

---

## Example Pseudo Code (Before Spring Boot Actuator)
----------------------------------------------------

```java
class HealthServlet extends HttpServlet {
    public void service(HttpRequest req, HttpResponse res) {
        // try connecting to database
        throw exception;

        // try checking external gateways and endpoints
        throw exception;
    }
}
````

* The above custom code **does not need to be written** anymore because Spring Boot Actuator provides these endpoints out of the box.

* Metrics and monitoring endpoints required for production readiness are provided automatically through **Actuator Endpoints**.

## Pictorial Respenstation of Acutator Endpoints 

 ![acutator_enpoints](https://github.com/user-attachments/assets/74100951-7470-4cb7-a350-9e11550ce0ce)


# Spring Boot CLI
--------------------

- When a task is assigned in a tech company, before implementing it fully, we usually:  
  - Write the **pseudo code**  
  - Create a **POC (Proof of Concept)**  

- To study the requirements and understand the technical implementation, sometimes we need to create pseudo code or POCs before starting the real design and development.

- Even though we can write pseudo code inside the actual project, after experimentation we need to clean up that code.  
  This takes time and also carries the **risk of breaking or introducing bugs** into the main project.

- Many times, to avoid such risks, we create a **separate project** just for pseudo code or POC development.  
  However, this also requires time and effort to set up.

- To solve this problem, **Spring Boot CLI** was introduced.

---

## Example Code
----------------

### `HomeController.java`
```java
@Controller
@RequestMapping("/home")
class HomeController {
    public String showHomePage() {
        return "home";
    }
}
````

### To run this code normally, we need:

i. Create the project and configure dependencies
ii. `web.xml`
iii. DispatcherServlet configuration
iv. Handler Mapping
v. View Resolver
vi. Package into WAR
vii. Deploy on application server

---

### Spring Boot CLI Advantage

Spring Boot CLI is a **command-line tool**, and we do **not** need to perform all the above steps.

You can simply run:

```
SpringApplication.run HomeController.java
```

Using Spring Boot CLI makes experimentation, pseudo code, and POC development extremely fast and effortless.


### Spring Boot Arthitecute

  ![WhatsApp Image 2025-11-19 at 14 43 56_a341278c](https://github.com/user-attachments/assets/fc748436-232c-4371-ba22-51f911258a25)
    
  
- Spring Boot is a tool that enables developers to work with the Spring Framework easily.
```

