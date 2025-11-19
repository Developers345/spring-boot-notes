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


