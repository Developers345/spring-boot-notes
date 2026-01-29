# Association Mapping Model
--------------------------

## What is Association Mapping?

Association Mapping Model explains **how to map the association relationship between entity classes into relational database tables**.

---

## What is an Association Relationship?

An **association relationship** between classes is established by **declaring one class as an attribute inside another class**.

We declare a class as an attribute in another class in order to **reuse the data or functionality** of the other class.

Association relationship always represents a **"HAS-A"** relationship between classes.

### Example

```java
class Person {
    Address address; // HAS-A relationship
}
````

---

## Characteristics of Association Relationship

In an association relationship, there are **two main characteristics**:

1. **Cardinality**

   * Defines **how many occurrences** of another object a class can hold.

2. **Directionality**

   * Defines whether the relationship is:

     * **Unidirectional**
     * **Bidirectional**

---

## Types of Association Relationships (Object World)

A class can be in an association relationship with another class as:

* **1–1** → One-to-One association
* **1–*** → One-to-Many association
* ***–*** → Many-to-Many association

---

## Association in the Relational World

In **relational databases**, we cannot directly express association relationships between tables.

The **only way** to represent relationships between data in the relational world is by using:

* **Primary Keys**
* **Foreign Keys**

These relationships are inherently **unidirectional**.

### Common Relational Mappings

* **One-to-One**
* **One-to-Many / Many-to-One**
* **Many-to-Many**

---

## Association Mapping Model

The process of representing **association relationships of entity classes** into **database relationships** is called the **Association Mapping Model**.

In this model:

* Object associations are mapped using **foreign keys**, **join tables**, or **shared primary keys**.
* Each type of object association must be carefully mapped into an equivalent relational structure.
