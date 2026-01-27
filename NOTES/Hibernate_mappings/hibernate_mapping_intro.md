# Entity Relationship Mapping

An **Entity object** can have a relationship with another entity object in the real world.  
The key question is: **how do we model (or) represent the relationship between these entity classes in Java into corresponding database tables**, so that we can perform operations on the objects based on their relationships?

**In short:**  
How to model **Entity Objects** with their relationships into **Relational Database table relationships**.

---

## Example Entity Classes

```java
class Customer {
  int customerNo;
  String customerName;
  int age;
  String gender;
  String mobileNo;
  Set<Sale> sales;
  // accessors
}

class Sale {
  int saleNo;
  Date saleDate;
  int quantity;
  double amount;
  // accessors
}
````

### Sample Object Representation

```json
{
  "customerNo": 1,
  "customerName": "smith",
  "age": 51,
  "gender": "Male",
  "mobileNo": 93983894,
  "sale": {
    "saleNo": 1,
    "saleDate": "12/05/2021",
    "quantity": 10,
    "amount": 3838
  }
}
```

---

## Understanding the Relationship

The `Customer` and `Sale` classes are in a **Has-A relationship**, indicating **which customer has made which sales**.

**Entity Relationship Mapping** focuses on:

* Designing database tables that represent these entity classes
* Establishing proper table relationships
* Ensuring object relationships are correctly persisted and retrieved

### Key Requirements

1. **Persistence**

   * When we persist a `Customer` object, both **customer data** and **associated sales data** should be stored.
   * The **association relationship** must also be persisted in database tables.

2. **Retrieval**

   * When fetching a customer, we should also be able to fetch their **associated sales**.
   * The relationship should allow seamless navigation from `Customer` to `Sale`.

---

## How Many Ways Can Objects Be in Relationship?

There are **two primary ways** an object can be in a relationship with another object.

---

## 1. Inheritance

Inheritance represents an **Is-A relationship** between classes.

* Used when all traits of a superclass are common to a subclass
* Enables reuse of existing data and functionality

### Characteristics

* A superclass can sometimes be **abstract** (Generalization)
* **Polymorphic access** is supported
* Any subclass can be referenced using a superclass type

```
    A
   / \
  B   C
```

```java
A a = new B();
```

---

## 2. Association

Association represents a **Has-A relationship** between classes.

* One class holds a reference to another class
* Used to reuse data and functionality without inheritance

### Characteristics

* Can be **Uni-Directional** or **Bi-Directional**
* **Cardinality** of relationship exists (one-to-one, one-to-many, etc.)

---

## Relational Model

In relational databases, **one table can be related to another table** in the following ways:

1. **One-to-One**
2. **One-to-Many**
3. **Many-to-Many**

### Characteristics

* Table relationships are **uni-directional**
* Relationships are established using **primary keys and foreign keys**

---

## Types of Entity Relationship Mapping Models

There are **two major types** of Entity Relationship Mapping models:

1. **Inheritance Mapping Model**
2. **Association Mapping Model**

---

## Pictorial Representation
<img width="3568" height="4172" alt="12-JUN-2021-DBMAPPINGg" src="https://github.com/user-attachments/assets/a2d6ba69-6f38-4e55-814e-89e49eb3d340" />
