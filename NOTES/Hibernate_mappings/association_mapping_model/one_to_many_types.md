# Collection Types Supported by Hibernate/JPA for One-to-Many Association

Hibernate/JPA supports **three types of collections** to model a **One-to-Many** association between entity classes.

1. **Set**
2. **List**
3. **Map**

---

## 1. Set

A **Set** is used when we want to store **unique child objects** for a given parent entity.

### Characteristics

- Stores only unique child objects.
- Prevents duplicate entries.
- Uses `equals()` and `hashCode()` methods to determine uniqueness.
- Does **not preserve** insertion order.

### When to Use

Use **Set** when:

- Duplicate child entities should not be allowed.
- The order of child entities is not important.

### Example

```java
@OneToMany(mappedBy = "department")
private Set<Employee> employees = new HashSet<>();
```

---

## 2. List

A **List** is used when we want to **preserve the insertion order** of child entity objects.

### Characteristics

- Preserves insertion order.
- Allows duplicate elements.
- Supports index-based access.

### When to Use

Use **List** when:

- Child entities should be retrieved in the same order they were inserted.
- Duplicate child entities are acceptable.

### Example

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees = new ArrayList<>();
```

---

## 3. Map

A **Map** is used when we want to **store additional information (key-value pairs)** along with the relationship.

### Characteristics

- Stores data as **key-value pairs**.
- Useful when each child entity needs to be accessed using a unique key.
- Can hold additional data outside the relationship itself.

### When to Use

Use **Map** when:

- Additional metadata needs to be associated with child entities.
- Child entities need to be accessed using a unique key.

### Example

```java
@OneToMany(mappedBy = "project")
@MapKey(name = "memberNo")
private Map<Integer, Member> members = new HashMap<>();
```

---

# Example Entity Classes

## Member Entity

```java
public class Member {

    private int memberNo;
    private String memberName;
    private String projectName;
    private Date startDate;
    private String role;
    private int experience;

}
```

## Task Entity

```java
public class Task {

    private int taskNo;
    private String title;
    private String description;
    private int duration;
    private String status;

}
```

---

# Comparison of Collection Types

| Feature | Set | List | Map |
|---------|-----|------|-----|
| Stores Unique Elements | ✅ Yes | ❌ No | ✅ Keys are unique |
| Preserves Insertion Order | ❌ No | ✅ Yes | Depends on Map implementation |
| Allows Duplicate Values | ❌ No | ✅ Yes | Keys: ❌, Values: ✅ |
| Index-Based Access | ❌ No | ✅ Yes | ❌ No |
| Key-Based Access | ❌ No | ❌ No | ✅ Yes |
| Best Use Case | Avoid duplicate child entities | Preserve insertion order | Store child entities with additional key-value information |

---

# Summary

- **Set** → Use when only **unique child entities** are required.
- **List** → Use when **insertion order** of child entities should be preserved.
- **Map** → Use when **additional key-value information** needs to be maintained along with the association.

  
# Pictorial Representation of one-to-many list
<img width="3568" height="4172" alt="29-JUN-2021-ONE-TO-MANYLIST" src="https://github.com/user-attachments/assets/ada7d462-9002-43ed-a6ed-46e10af29af4" />

# One-to-Many Association Using Map Collection

A **Map** collection is used in a **One-to-Many** association when there is **additional data (or columns)** that needs to be maintained as part of the relationship between the parent and child entities.

Unlike **Set** and **List**, which simply store collections of child entities, a **Map** allows you to associate each child entity with a **unique key**. This is useful when extra information is generated **because of the relationship itself**.

---

## When to Use a Map Collection

Use a **Map** in a One-to-Many association when:

- Additional data needs to be stored along with the relationship.
- The additional data exists **only because the relationship exists**.
- Child entities need to be accessed using a unique key.

---

## Example Scenario

Consider a **WorkOrder** and a **Contractor**.

- A **WorkOrder** can be assigned to a **Contractor**.
- When the assignment takes place, a **Billing Number** or **Contract Number** is generated.
- The **Billing Number** or **Contract Number** has **no existence** until the work order is assigned to the contractor.

Since this information is **generated as a result of the relationship**, it is considered **relationship-specific data** and can be maintained using a **Map-based association**.

---

## Illustration

```text
WorkOrder  -------------------->  Contractor
     │
     │ Assignment
     ▼
Billing Number / Contract Number
```

Here:

- **Parent Entity:** WorkOrder
- **Child Entity:** Contractor
- **Relationship Data:** Billing Number / Contract Number

---

## Example Mapping

```java
@OneToMany(mappedBy = "workOrder")
@MapKey(name = "billingNo")
private Map<String, Contractor> contractors = new HashMap<>();
```

In this example:

- The **key** is the `billingNo`.
- The **value** is the `Contractor` object.
- Each contractor can be retrieved using its corresponding billing number.

---

## Advantages of Using Map

- Stores relationship-specific data efficiently.
- Enables fast lookup using a unique key.
- Represents key-value relationships naturally.
- Suitable when additional information is generated during the association.

---

## Summary

- **Set** → Use when child entities must be **unique**.
- **List** → Use when the **insertion order** of child entities must be preserved.
- **Map** → Use when **additional relationship-specific data** (such as Billing Number, Contract Number, etc.) needs to be stored along with the association.

# Pictorial Representation of one-to-many map
<img width="3568" height="4172" alt="30-JUN-2021-ONE-TO-MANY-MAP" src="https://github.com/user-attachments/assets/b77a117f-6e56-456a-ad9a-cd37ef0de2cc" />

