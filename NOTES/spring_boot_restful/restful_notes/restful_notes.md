# Restful Services

## Introduction
- **Restful Services** expose the business components of an application in a **distributed** and **interoperable** manner.
- The business components implemented based on **RESTful principles** are called **Restful Services**.

### Key Terms
- **Distributed**: Something that can be accessed over a network.
- **Interoperable**: A component in a web application can be accessed by applications built using any programming language and running on any platform (OS).

---

## What are Restful Services?  
## What is the purpose of Restful Services?

Restful Services expose the business components of an application to **external business partners** in a **distributed** and **interoperable** manner.

---

## Pictrial Respresentation -
<img width="1897" height="660" alt="Screenshot 2026-01-12 071637" src="https://github.com/user-attachments/assets/3feee9e9-3561-45d0-b2b5-24d447ff3596" />

## Restful Services vs SOAP Web Services

In the case of **SOAP Web Services**, they are purely used for building **integration solutions**.  
That means when two different, disparate, heterogeneous applications want to communicate with each other, SOAP services are used.

Unlike SOAP services, **Restful Services** are used in two ways:
1. **Integration Solutions**
2. **Backend Tier for Web Applications**

---
## Pictorial representation of Restful Service usecase 
<img width="3177" height="1336" alt="USECASE_OF_REST" src="https://github.com/user-attachments/assets/d63baf30-44dd-45e1-a9a3-f43f6c5bc441" />

## SOAP Web Services – Drawbacks
- Quite complex to use
- Poor performance
- Not easy to adopt
- Not as interoperable as they claim

---

## Why is the Web so Successful?

- How is the Web so scalable?
- What makes it easy to adopt?
- What principles does the Web follow?

These questions lead to an important idea:

> If we adopt the same principles on which the Web works while building business components,  
> then those components will also be **scalable**, **adoptable**, and **successful**.

# Restful Principles

## 1. Unique Addressable URI

Every component must have a **directly accessible, addressable URI** through which the component can be accessed.  
This makes the components **highly adoptable** and easy to use by different clients.

---

## Note

- A **Servlet** is a distributed component of a web application and is accessed using a **directly accessible addressable URI**.

- Before Restful Services, **SOAP Web Services** were widely used.  
  One major problem with SOAP is that it does **not** provide a directly accessible, addressable URI for each operation.

  In SOAP, requests are sent using:
  - an **endpoint URL**
  - along with an **action (method)**

  **Example:**
```

endpointurl: /insurance
action: enroll

```

- Another major issue with SOAP is that developers using other technologies (such as **.NET**, **SAP**, etc.) must have prior knowledge of **SOAP Web Services** to send requests.

- In contrast, **Restful Services** allow components to be accessed using **directly accessible URIs**, such as:
```

/insurance/enroll
/insurance/renewal

```

- Because of this URI-based access, developers from **any technology stack** can easily access Restful endpoints without needing special protocol knowledge.

## Pictorial representation - Unique Addressable URI 
<img width="1251" height="503" alt="Screenshot 2026-01-12 091941" src="https://github.com/user-attachments/assets/1f5671da-0974-4342-889f-e5dcff8a1158" />

## 2. Uniform Constrained Interfaces

A **fixed set of methods** must be used across all components while developing services.  
This uniformity makes the system **easy to understand, use, and adopt** by different clients.


## Pictorial representation - Uniform Constrained Interfaces
<img width="1840" height="595" alt="Screenshot 2026-01-12 093853" src="https://github.com/user-attachments/assets/76387760-ae36-4c76-8443-83f7927f9f41" />

---

## 3. Communication Stateless

- The server should **never store any client-specific state** on behalf of the client  
  (for example, no `HttpSession` objects).
- Each request from the client must contain **all the information** required to process it.
- This approach helps in better utilization of computing resources by accommodating a **large number of users**.
- As a result, the application becomes **highly scalable**.

## Pictorial representation - Communication Stateless
<img width="1259" height="587" alt="Screenshot 2026-01-12 094658" src="https://github.com/user-attachments/assets/cc976bad-17f1-4dc4-94e9-e4c5ea1cccef" />

---

## 4. Representation Oriented

- The client can choose **any data representation format** supported by the service to communicate.
- Common representations include:
  - JSON
  - XML
  - YAML
- The client is **not restricted to a single representation format**, making the service more flexible and interoperable.

 ## Pictorial representation - Representation Oriented
 <img width="1341" height="606" alt="Screenshot 2026-01-12 095112" src="https://github.com/user-attachments/assets/75f93f53-d6f7-4f1f-81c5-0c23ea7524fb" />

---

## 5. HATEOAS  
**Hypermedia as an Engine of Application State**

- For every interaction with the service, along with the response data, the service should return a **set of hyperlinks**.
- These hyperlinks help the client understand:
  - What **possible actions** can be performed next
  - How to navigate through the application state
- This makes the client less dependent on hard-coded URLs and improves **discoverability** of the API.
  
## Pictorial Representation of all Rest principles
<img width="3177" height="1336" alt="REST-PRINCIPLES" src="https://github.com/user-attachments/assets/f03e09b2-9e22-4cda-a060-91c7bfad0802" />
