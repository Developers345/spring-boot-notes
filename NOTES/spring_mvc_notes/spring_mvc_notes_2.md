# Spring MVC Architecture

<img width="1018" height="603" alt="image" src="https://github.com/user-attachments/assets/e15f6395-e1da-409e-9548-e53f26830c7c" />

<img width="1889" height="746" alt="Screenshot 2026-01-06 125606" src="https://github.com/user-attachments/assets/61215a5f-960a-404d-a3f6-70fcca3c596c" />

# Spring MVC Flow

<img width="1030" height="665" alt="Screenshot 2026-01-06 151106" src="https://github.com/user-attachments/assets/db347766-efca-49a7-8c0f-97ed81f70d98" />

# Explaination of Spring MVC Flow

- In the case of Spring MVC, the user has to send the request with an extension such as  
  `*.htm`, `*.mvc`, `*.web`, or `*.action`, depending on the **DispatcherServlet**
  configuration wildcard pattern.

- When the user sends a request, it is received by the **DispatcherServlet**.
  The DispatcherServlet then asks the **HandlerMapping** to provide controller
  information by passing the incoming URL.

- The **HandlerMapping** takes the URL, maps it to the appropriate **Controller**,
  and returns the controller information to the DispatcherServlet.

- The **DispatcherServlet** forwards the request to the **Controller** for processing.

- The **Controller** processes the request and returns a **logical view name**
  back to the DispatcherServlet.

- After receiving the logical view name, the **DispatcherServlet** asks the
  **ViewResolver** to resolve the view by passing the logical view name.

- The **ViewResolver** takes the logical view name, creates the appropriate
  **View object**, and returns it to the DispatcherServlet.

- Finally, the **DispatcherServlet** renders the resolved view and sends the
  response back to the client.
```
