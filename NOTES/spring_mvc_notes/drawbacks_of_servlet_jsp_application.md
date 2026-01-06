# Servlet and JSP

## Why we should not write the business logic and persistence logic inside the servlet?

- Servlets are **container components**, and if we write business logic and persistence logic inside them, we **cannot reuse** that logic elsewhere in the application.
- Servlet is a **presentation-tier component**. If we switch the presentation-tier technology (Struts, Spring MVC, etc.), we need to **rewrite the business and persistence logic** in another class.

---

## Problems with Servlet Applications

### 1. Request Handling and Request Wrapping
- For various requests in an application, we have **different servlets**.
- For each request, to hold its data, we need a **form class**.
- A servlet is responsible for:
  - Receiving the request
  - Reading data from the HTTP request
  - Binding the data to a form object
- The process of binding request data to a form object is called **Request Wrapping**.
- In all servlets of the application, we need to write request-wrapping logic, which becomes **boilerplate code**.

> **Note:**  
> - Form object is **specific to the request**.  
> - For every request, **one form object** will be created.

---

### 2. No Central Navigation or View Management
- There is **no central navigation management or view management** available in Servlet/JSP-based applications.
- It is difficult to determine:
  - Which requests are forwarding to which pages
- Changing a view/page requires **a lot of effort**, as we need to identify all the places where it is referenced.

---

### 3. No Built-in Validation Support
- There is **no built-in support** for:
  - Data validations
  - Returning the view back to the user with **validation error messages**

---

### 4. No Centralized Request Control
- We do not have **central control** over receiving and processing requests.

---

## Note

- **Filters** are used to **decorate requests and responses**,  
  **not** for conditional checks of requests.

## Pictorial representation 
<img width="1919" height="612" alt="Screenshot 2026-01-06 104946" src="https://github.com/user-attachments/assets/5ab8dea5-ad79-4b94-ae83-ee780424bc45" />

