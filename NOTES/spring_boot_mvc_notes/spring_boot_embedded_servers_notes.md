# Spring Boot MVC – Embedded Servlet Containers

-> In Spring Boot, we can run a Spring MVC web application as a **JAR file itself**, not as a WAR file.  
Spring Boot uses a concept called **Embedded Servlet Containers**.

---

## Embedded Servlet Containers

---

## Problems with Traditional Development and Deployment of Web Applications

1. Delivering code alone does not make the application work.  
   We need to download and set up **server runtimes** to deploy and run the application, which delays bringing the application up.

2. To run a web application, we need both **application configuration** and **server runtime configuration**, such as:
   - Changing port numbers for application servers (WebLogic, GlassFish)
   - DataSource configurations like managing the connection pool
   - Configuring security rules on application servers (WebLogic, GlassFish), etc.

   Even though these are part of the application requirements, we must explicitly apply server runtime configurations on the server before running the application.

3. Debugging takes more time because the server runtime must be restarted repeatedly after code changes.

4. Not suitable for **microservice-based deployments**.

---
## Pictorial Presentation 
<img width="1894" height="550" alt="Screenshot 2026-01-09 175457" src="https://github.com/user-attachments/assets/3709c79b-7cf7-4abf-961c-475440fe175f" />


## Advantages of Embedded Server Development and Deployment

-> Embedded containers are **lightweight JAR distributables** included as part of the Java project.

-> Every Spring Boot web application has **one isolated servlet container**.  
If two Spring Boot web applications are running, then **two separate servlet containers** will be running—one for each application.

---

## Embedded Servers Supported by Spring Boot

- Tomcat  
- Jetty  
- Netty  

---

## Key Benefits

1. We can run the application directly from the source code without downloading or setting up any external server runtime.
2. Server runtime configurations are applied through code execution, so no separate server configuration is required before deployment.
3. Debugging takes less time because application packaging is avoided.
4. Perfectly suitable for **microservice-based applications**.

---

## Internal Working

-> Spring Boot internally creates a **runtime WAR (logical WAR)** and places it inside the **embedded Tomcat server**.

## Pictorial Presentation 
<img width="1873" height="547" alt="Screenshot 2026-01-09 180338" src="https://github.com/user-attachments/assets/b857da5b-9850-41c9-958d-6013190c8437" />
