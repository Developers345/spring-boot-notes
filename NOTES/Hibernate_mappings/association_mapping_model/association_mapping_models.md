# One-to-Many Association in Hibernate/JPA

In a **One-to-Many** association, one side of the relationship is declared as a **collection of objects** belonging to another entity class. This represents that **one parent entity can have multiple child entities**.

Hibernate/JPA supports the following collection types to represent association relationships:

## 1. Set

A **Set** is a collection that stores multiple objects while ensuring that only **unique** objects are present.

### Characteristics

- Stores multiple objects.
- Allows **only unique** elements.
- Determines uniqueness using:
  - `equals()`
  - `hashCode()`
- Does **not preserve** the insertion order of elements.
- When retrieving elements, the order is **not guaranteed**.

### When to Use

Use a **Set** in a One-to-Many association when:

- Duplicate child objects should **not** be allowed.
- The order of child objects is **not important**.

### Example

```java
@OneToMany(mappedBy = "department")
private Set<Employee> employees = new HashSet<>();
```

---

## 2. List

A **List** is an **index-based** collection that stores multiple objects while preserving their insertion order.

### Characteristics

- Stores multiple objects.
- Preserves the **insertion order** of elements.
- Allows duplicate elements.
- Elements can be accessed using their index.

### When to Use

Use a **List** in a One-to-Many association when:

- The insertion order of child objects must be preserved.
- Duplicate child objects are acceptable.

### Example

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees = new ArrayList<>();
```

---

## Comparison: Set vs List

| Feature | Set | List |
|---------|-----|------|
| Duplicate Elements | ❌ Not Allowed | ✅ Allowed |
| Preserves Insertion Order | ❌ No | ✅ Yes |
| Index-Based Access | ❌ No | ✅ Yes |
| Uses `equals()` & `hashCode()` | ✅ Yes | ❌ No |
| Best Use Case | Avoid duplicate child entities | Preserve insertion order of child entities |

---

## Summary

- Use **Set** when child entities must be **unique** and ordering is not required.


## Pictorial Representation - many-to-one association model
<img width="3568" height="4172" alt="27-JUN-2021-MANY-TO-ONE" src="https://github.com/user-attachments/assets/cfeef3c8-a192-417e-bc06-99324f708bb2" />


## Pictorial Representation - many-to-one association model - 1
<img width="3568" height="4172" alt="28-JUN-2021" src="https://github.com/user-attachments/assets/05c214c2-95f6-425a-be92-484e3510636d" />
- Use **List** when the **order of child entities matters**, even if duplicates are allowed.
````
