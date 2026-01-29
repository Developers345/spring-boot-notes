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
* Another example is table_per_class hirarchy in inheritence mapping model.

Managing this difference in structure is called **Granularity Mis-Match**.


