# Spring MVC / Spring Boot REST Request Handling – Detailed Explanation

# Pictorial Respresentation - modeling spring rest
<img width="1620" height="717" alt="Screenshot 2026-01-12 160811" src="https://github.com/user-attachments/assets/51c31ee9-1b13-41cd-8c62-a21fa5aa13b3" />

This document explains the concepts shown in the screenshot related to **HTTP request structure** and **how Spring MVC / Spring Boot maps request data to controller methods**.

---

## 1. HTTP Request Structure

An HTTP request sent from a client (browser, Postman, frontend app) consists of **three main parts**:

### 1.1 Request Line / URI
- Contains:
  - **HTTP method** (GET, POST, PUT, DELETE)
  - **URI (URL path)**
  - **Query parameters**

Example:
```

GET /netbanking/url/accounts?id=10&type=savings

````

---

### 1.2 Headers
Headers carry **metadata** about the request.

Examples:
- `Content-Type: application/json`
- `Authorization: Bearer <token>`
- `Accept: application/json`
- Cookies (sent internally as headers)

---

### 1.3 Body
The request body contains **actual data** sent to the server.

Supported formats:
- JSON
- XML
- YAML
- Form data

Example JSON body:
```json
{
  "accountHolderName": "Mike",
  "age": 34,
  "gender": "Male"
}
````

---

## 2. Spring REST Controller Basics

### 2.1 `@RestController`

```java
@RestController
class Resource {
}
```

* Combination of:

  * `@Controller`
  * `@ResponseBody`
* Automatically converts return values to **JSON / XML**
* Used for REST APIs

---

### 2.2 Base URL Mapping

```java
@RequestMapping("/netbanking")
class Resource {
}
```

* Acts as **base path** for all APIs inside the controller
* Example final URL:

```
/netbanking/url/subUrl
```

---

## 3. Method-Level Request Mapping

### 3.1 Mapping HTTP Methods

```java
@GetMapping("/subUrl")
@PostMapping("/subUrl")
@PutMapping("/subUrl")
@DeleteMapping("/subUrl")
```

Equivalent to:

```java
@RequestMapping(value="/subUrl", method=RequestMethod.GET)
```

---

### 3.2 Sample Controller Method

```java
@GetMapping("/subUrl")
public Object resourceMethod(parameters) {
    return object;
}
```

* Handles incoming request
* Returns response data
* Response automatically converted to JSON

---

## 4. Handling Query Parameters

### 4.1 Query Parameter Syntax

URL:

```
/accounts?type=savings&limit=5
```

Controller:

```java
@GetMapping("/accounts")
public String getAccounts(
    @RequestParam("type") String type,
    @RequestParam("limit") int limit
) {
    return "OK";
}
```

* `@RequestParam` extracts **query parameters**
* Optional parameters:

```java
@RequestParam(required = false)
```

---

## 5. Handling Path Parameters

### 5.1 Path Variable Syntax

URL:

```
/accounts/101
```

Mapping:

```java
@GetMapping("/accounts/{accountId}")
public String getAccount(
    @PathVariable("accountId") int id
) {
    return "Account ID: " + id;
}
```

### Key Points:

* `{accountId}` defines **position** in URL
* `@PathVariable` extracts value from URL

---

## 6. Handling Headers

```java
@GetMapping("/header")
public String readHeader(
    @RequestHeader("Authorization") String token
) {
    return token;
}
```

* Used for:

  * Authentication tokens
  * Custom metadata

---

## 7. Handling Cookies

```java
@GetMapping("/cookie")
public String readCookie(
    @CookieValue("JSESSIONID") String sessionId
) {
    return sessionId;
}
```

* Reads cookie values sent by client

---

## 8. Binding Request Body to Object

### 8.1 Using `@RequestBody`

```java
@PostMapping("/account")
public String createAccount(@RequestBody Account account) {
    return "Created";
}
```

### 8.2 Account POJO

```java
public class Account {
    private String accountHolderName;
    private int age;
    private String gender;
}
```

* JSON automatically converted to Java object
* Uses **Jackson** internally
* Content-Type must be `application/json`

---

## 9. Complete Example

```java
@RestController
@RequestMapping("/netbanking")
public class Resource {

    @PostMapping("/account/{id}")
    public Account createAccount(
        @PathVariable int id,
        @RequestParam String type,
        @RequestHeader("Authorization") String token,
        @RequestBody Account account
    ) {
        return account;
    }
}
```

---

## 10. Summary Table

| Request Part | Spring Annotation                     |
| ------------ | ------------------------------------- |
| Query Param  | `@RequestParam`                       |
| Path Param   | `@PathVariable`                       |
| Header       | `@RequestHeader`                      |
| Cookie       | `@CookieValue`                        |
| Body         | `@RequestBody`                        |
| URL Mapping  | `@RequestMapping`, `@GetMapping`, etc |

---

## 11. Key Takeaways

* HTTP request consists of **URI + Headers + Body**
* Spring MVC maps each part using **annotations**
* `@RestController` simplifies REST API development
* Automatic data binding and serialization reduces boilerplate code
* Clean separation of request data handling improves maintainability

---

## Message Converters & `@RestController` – Detailed Explanation

This section expands on the given points and explains **how Spring MVC / Spring Boot internally handles data conversion and responses**.

---

## 1. Message Converters (HttpMessageConverter)

### 1.1 What is a Message Converter?

**Message Converters** are Spring MVC components responsible for:

- Converting **request body data** (JSON / XML / YAML)  
  ➜ into **Java objects**
- Converting **Java objects**  
  ➜ back into **response body data** (JSON / XML / YAML)

This conversion happens **automatically** in REST controllers.

> Internally, Spring uses the interface:
```java
HttpMessageConverter<T>
````

---

## 1.2 Why Message Converters Are Needed?

HTTP requests and responses **cannot directly understand Java objects**.

So Spring needs:

* A way to **read** incoming data formats
* A way to **write** outgoing data formats

Message converters act as a **bridge** between:

```
Client (JSON/XML/YAML) ⇄ Spring MVC ⇄ Java Objects
```

---

## 1.3 Common Built-in Message Converters

Spring Boot auto-configures converters based on classpath dependencies.

| Data Format | Converter Class                                 |
| ----------- | ----------------------------------------------- |
| JSON        | `MappingJackson2HttpMessageConverter`           |
| XML         | `MappingJackson2XmlHttpMessageConverter` / JAXB |
| YAML        | `YamlHttpMessageConverter`                      |
| Text        | `StringHttpMessageConverter`                    |
| Byte data   | `ByteArrayHttpMessageConverter`                 |
| Form data   | `FormHttpMessageConverter`                      |

👉 **Jackson** is used by default for JSON conversion.

---

## 1.4 Request Flow Using Message Converters

### Example Request

```http
POST /account
Content-Type: application/json
```

```json
{
  "accountHolderName": "Mike",
  "age": 34,
  "gender": "Male"
}
```

### Controller Code

```java
@PostMapping("/account")
public String createAccount(@RequestBody Account account) {
    return "Success";
}
```

### Internal Flow

1. DispatcherServlet receives request
2. Detects `@RequestBody`
3. Finds suitable `HttpMessageConverter`
4. Converts JSON → `Account` object
5. Passes Java object to controller method

---

## 1.5 Response Flow Using Message Converters

### Controller Return

```java
@GetMapping("/account")
public Account getAccount() {
    return new Account("Mike", 34, "Male");
}
```

### Internal Flow

1. Controller returns Java object
2. `@ResponseBody` triggers response handling
3. Message Converter selected based on:

   * `Accept` header
   * Available converters
4. Java Object → JSON/XML/YAML
5. Response sent to client

---

## 1.6 Content Negotiation

Spring decides **which format to use** based on:

* `Content-Type` (request)
* `Accept` (response)
* Available message converters

Example:

```http
Accept: application/xml
```

Response will be in **XML** if XML converter is available.

---

## 1.7 Summary of Message Converters

* Automatic conversion
* Pluggable and extensible
* Eliminates manual parsing
* Central to REST API development

---

## 2. `@ResponseBody` Annotation

### 2.1 Purpose of `@ResponseBody`

```java
@ResponseBody
public Account getAccount() {
    return account;
}
```

* Tells Spring:

  > "Do NOT resolve a view, write this object directly to the HTTP response body"

Without `@ResponseBody`:

* Spring treats return value as **view name**

---

## 2.2 Problem Without `@ResponseBody`

```java
@Controller
public class TestController {

    @GetMapping("/test")
    public Account test() {
        return new Account();
    }
}
```

❌ Spring tries to find a view named `Account.jsp`

---

## 3. `@RestController` Explained

### 3.1 What is `@RestController`?

```java
@RestController
public class ResourceController {
}
```

`@RestController` is a **composed annotation**:

```java
@RestController
= @Controller + @ResponseBody
```

---

## 3.2 Why `@RestController` Is Important?

* Eliminates need to write `@ResponseBody` on **every method**
* Cleaner and readable code
* Designed specifically for **REST APIs**

### Equivalent Code

```java
@Controller
@ResponseBody
public class ResourceController {
}
```

---

## 3.3 Method-Level Implicit Behavior

When using `@RestController`:

```java
@GetMapping("/account")
public Account getAccount() {
    return account;
}
```

Internally behaves as:

```java
@GetMapping("/account")
@ResponseBody
public Account getAccount() {
    return account;
}
```

---

## 4. How Message Converters Work with `@RestController`

### Combined Flow

```
Client Request (JSON)
   ↓
DispatcherServlet
   ↓
@RequestBody
   ↓
HttpMessageConverter
   ↓
Java Object
   ↓
@RestController
   ↓
@ResponseBody
   ↓
HttpMessageConverter
   ↓
JSON Response
```

---

## 5. Key Takeaways

* **Message Converters** handle all data format transformations
* They convert:

  * JSON/XML/YAML ⇄ Java Objects
* `@ResponseBody` tells Spring to write data directly to HTTP response
* `@RestController` implicitly applies `@ResponseBody` to all methods
* Together they form the **foundation of Spring REST APIs**

---
Idempotent
----------

**Idempotent** means:

> If the same request is sent **once or multiple times**, the **final effect on the server resource remains the same**.

In other words, repeating an idempotent request does **not create additional or unintended side effects** on the resource.

---

### Why Idempotency Matters
- Helps in **network failure recovery**
- Safe for **retries** (client can resend the request if it is unsure whether the first request succeeded)
- Ensures **data consistency**

---

### Idempotent HTTP Methods

The following HTTP methods are **idempotent** because the client sends the request with a **resource identifier (ID)**, allowing the server to clearly understand whether the request is new or a repeat.

#### 1. GET
- Used to **read** data
- Does **not modify** the resource
- Sending it multiple times returns the same result

**Example**
```http
GET /users/101
````

➡️ Fetches user details every time, without changing anything.

---

#### 2. PUT

* Used to **create or completely replace** a resource
* Repeating the same PUT request results in the **same final state**

**Example**

```http
PUT /users/101
{
  "name": "Gireesh",
  "role": "Admin"
}
```

➡️ No matter how many times this request is sent, user `101` will have the same data.

---

#### 3. DELETE

* Used to **remove** a resource
* Deleting an already deleted resource has **no additional effect**

**Example**

```http
DELETE /users/101
```

➡️ After deletion, repeating the request still results in the resource being deleted.

---

### Non-Idempotent HTTP Method

#### POST

* **POST is NOT idempotent**
* Each request usually **creates a new resource**
* Sending the same POST request multiple times results in **multiple changes**

**Example**

```http
POST /users
{
  "name": "Gireesh"
}
```

➡️ Each request creates a **new user**, even if the data is identical.

---

### Summary Table

| HTTP Method | Idempotent | Reason                                |
| ----------- | ---------- | ------------------------------------- |
| GET         | ✅ Yes      | Read-only, no state change            |
| PUT         | ✅ Yes      | Replaces resource with same state     |
| DELETE      | ✅ Yes      | Removes resource (no repeated effect) |
| POST        | ❌ No       | Creates a new resource each time      |

---

### Simple One-Line Definition

> **Idempotent operations can be safely repeated without changing the final result.**




