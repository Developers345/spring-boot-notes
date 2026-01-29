# Impedance Mis-Match

Impedance Mis-Match talks about the difference that exists between the way data is represented and used in **objects** versus the **relational model**.

- **RDBMS** represents data in **table format**
- **Objects** store data as an **inter-connected object graph**

Because of this difference, when we try to **load and store objects** into **relational tables**, several problems arise.  
While mapping the **object model** to the **table (relational) model**, we encounter **five major problems**, which are collectively called **Impedance Mis-Match**.

If we can provide solutions for these impedance problems, then we can achieve **ORM (Object Relational Mapping)**.

## Types of Impedance Mis-Match

1. Granularity  
2. SubTypes  
3. Identity  
4. Association  
5. Navigatability  

---

## What is Impedance Mis-Match and How Do We Solve It?

Impedance Mis-Match is also called **Paradigm Mis-Match**.

When we try to map **objects** into the **relational model**, we find several differences between the **object-oriented world** and the **relational world**. These differences are termed as **Impedance Mis-Matches**.

If we are able to provide solutions for these mis-matches, then we can successfully achieve **ORM**.

There are **five major mis-matches** between the object world and the relational world.

---

## 1. Granularity Mis-Match

**Granularity Mis-Match** occurs when:

- The number of **classes** in the object model
- And the number of **tables** in the relational model  

are **not the same**.

The challenge is how to persist object data into relational tables when their structures differ.

### Example

#### Object Model

```java
class Person {
    Address address;
}

class Address {
}
````

#### Relational Model

```text
person
------
id,
name,
age,
gender,
address_line1,
address_line2,
city,
state,
zip,
country
```

In this case:

* Multiple **object classes** (`Person`, `Address`)
* Are stored in a **single relational table** (`person`)
* Another example solution is table_per_class hirarchy in inheritence mapping model.

Managing this difference in structure is called **Granularity Mis-Match**.

## 2. SubTypes

In the **object-oriented world**, we can have one class **inherit** from another class for the sake of **reusability**.

However, in the **relational model**, we cannot **extend one table from another table**.

So, the problem of **how to map objects that have inheritance relationships into the relational world** is known as the **SubTypes Mis-Match**.

---

## 3. Identity

In the **database world**, we use a **primary key** as the identity of a record.  
Using this identity, we determine what operation has to be performed (insert, update, delete).

In the **Java world**, an object has **two identities**:

1. `equals()`
2. `hashCode()`

Because of this difference, the identity of the **database world** and the **Java world** are **not the same**, which can lead to **incorrect operations** being performed.

### Example

Assume a record already exists in the database:

```text
person table contains a record with primary key = 10
````

Now we create a new Java object:

```java
// We have created a new object p2
Person p2 = new Person(10, "bob", 30);
```

**Principle:**

* If an object already exists → **update**
* If an object does not exist → **insert**

```java
Person ep = session.get(Person.class, 10);

if (ep.equals(p2) == false) {
    session.save(p2);
}

if (ep == p2) {
    session.save(p2);
}
```

Since both `equals()` and `hashCode()` fail to correctly compare `p2` with `ep`, Hibernate tries to **save** the object even though a record with the same primary key already exists. This results in an **exception**.

### Solution

To correctly match **database identity** with **object identity**:

* Always construct `equals()` and `hashCode()` **based only on the database identity (primary key)**.

---

## 4. Association

In **Java**, we can establish an **association relationship** between classes by defining one class as an attribute inside another class.

```java
class Person {
    Address address;
}
```

However, in the **relational world**, we cannot directly represent **object associations**.

So, the challenge of **modeling association relationships of classes in relational tables** is called the **Association Mis-Match**.

---

## 5. Navigatability

In the **object-oriented world**, we can navigate from one object to another using **accessor methods**, provided the objects are associated.

```java
class Person {
    int id;
    String name;
    Address address;
    // accessors
}
```

```java
Person p = session.get(Person.class, 10);
Address address = p.getAddress();
```

The question is:

> How do we fetch the `Address` object when it is stored in a **different database table**, but accessed using a **getter method**?

In the **relational world**, data is stored in **separate tables**, so navigating through objects is not straightforward.

Handling this object navigation across multiple tables is known as the **Navigatability Mis-Match**.

---

## Conclusion

To successfully map data between **Java objects** and the **relational world**, we must solve all **five Impedance Mis-Match problems**:

1. Granularity
2. SubTypes
3. Identity
4. Association
5. Navigatability

**Object Relational Mapping (ORM)** technologies provide solutions for these problems.

Frameworks such as **Hibernate**, **JPA**, and other ORM tools in the market adopt these solutions to bridge the gap between the object-oriented world and the relational world.


