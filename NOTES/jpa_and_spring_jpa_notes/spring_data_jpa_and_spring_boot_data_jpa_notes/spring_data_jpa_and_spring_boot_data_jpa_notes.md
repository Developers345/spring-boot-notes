# Spring Data JPA

## Example:

### Bike Entity
```java
@Entity
@Table(name = "bike")
public class Bike {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "bike_no")
    protected int bikeNo;

    @Column(name = "bike_name")
    protected String bikeName;

    @Column(name = "bike_color")
    protected String bikeColor;

    @Column(name = "bike_model")
    protected String bikeModel;

    // accessor methods
}
````

### BikeDao

```java
public class BikeDao {

    @PersistenceContext
    private EntityManager entityManager;

    public int saveBike(Bike bike) {
        entityManager.persist(bike);
        return bike.getBikeNo();
    }

    public int updateBike(Bike bike) {
        return entityManager.merge(bike);
    }

    public int deleteBike(Bike bike) {
        entityManager.remove(bike);
        return 1;
    }
}
```

---

### Staff Entity

```java
@Entity
@Table(name = "staff")
public class Staff {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "staff_no")
    protected int staffNo;

    protected String firstName;
    protected String lastName;
    protected int age;
    protected String gender;
    protected String designation;
    protected String mobileNo;
    protected String emailAddress;

    // accessor methods
}
```

### StaffDao

```java
public class StaffDao {

    @PersistenceContext
    private EntityManager entityManager;

    public int saveStaff(Staff staff) {
        entityManager.persist(staff);
        return staff.getStaffNo();
    }

    public int updateStaff(Staff staff) {
        return entityManager.merge(staff);
    }

    public int deleteStaff(Staff staff) {
        entityManager.remove(staff);
        return 1;
    }
}
```

---

## Problem with the Above Approach

* The DAO logic is **duplicated** for every entity class.
* CRUD operations (`save`, `update`, `delete`) look almost identical across DAOs.
* Maintaining and extending such code becomes difficult as the number of entities grows.

---

## Birth of Spring Data Module

* The same concern was raised by developers in Spring Framework community forums.
* This led to the idea of the **Spring Data Module**.

### Spring Data Sub-Modules

1. **Spring Data JPA**
2. **Spring Data MongoDB**
3. **Spring Data Cassandra**
4. Others (Redis, Elasticsearch, etc.)

---

## Spring Data JPA – Key Concept

* **Spring Data JPA works on the concept of runtime-generated proxies**
* These proxies automatically perform persistence operations.
* Developers only need to define **repository interfaces**, not DAO implementations.
* This eliminates boilerplate CRUD code and improves productivity.

# Proxy Design Pattern

## What are Proxies
- Proxies are **surrogates or substitutes** for original classes.
- Proxy classes look like the original class but **add extra functionality** on top of the existing behavior.
- The original class logic remains unchanged, while additional responsibilities are handled by the proxy.

---

## Why Do We Need Additional Functionality in a Proxy Class?

- Some classes **require additional behavior**, while others do not.
- Adding multiple `if` conditions inside the same class to handle different behaviors is a **bad design practice**.
- Proxy pattern helps in **separating concerns** and keeping classes clean and focused.

---

## Example

```java
public class ImageIo {

    public byte[] getImage(String inImageFileLocation) {

        // Go to File System and read the image as bytes

        if (platform.equals("android")) {
            // compression logic
        }

        return bytes;
    }
}
````

### Explanation

* The responsibility of `ImageIo` is to **read an image from the file system** and render it to:

  * an Android application
  * a browser-based web application
* Android applications have **limited memory**, so images should be compressed.
* Web applications generally do **not require compression**.

### Problem with This Approach

* Adding platform-specific `if` conditions inside the same class is not a good idea.
* Some classes need compression logic, while others do not.
* This leads to:

  * tightly coupled code
  * poor maintainability

> **Note:**
> Using `if` conditions is acceptable when the logic is generic (for example, compressing images larger than 2MB).
> But platform-specific behavior should not be mixed into the same class.

---

## Interface-Based Design

```java
public interface ImageIo {
    public byte[] getImage(String inImageFileLocation);
}
```

### Implementation with Compression Logic

```java
public class ImageIoImpl implements ImageIo {

    public byte[] getImage(String inImageFileLocation) {

        // Go to File System and read the image as bytes

        if (platform.equals("android")) {
            // compression logic
        }

        return bytes;
    }
}
```

### Separate Class for Compressed Images

```java
public class ZippedImageIo implements ImageIo {

    public byte[] getImage(String imageFileLocation) {

        // Read the image from file system
        // Apply compression logic

        return bytes;
    }
}
```

### Problem with This Design

* Code to read images from the file system is **duplicated** in multiple classes.
* Maintenance becomes difficult.

---

## Proxy Design Pattern Solution

* Create a **proxy class** that:

  * implements the same interface as the original class
  * holds a reference to the original object
  * delegates the call to the original object
  * adds extra functionality when required

---

## Proxy Class Example

```java
public class ZippedImageIoImpl implements ImageIo {

    private ImageIo imageIo;

    public ZippedImageIoImpl() {
        imageIo = new ImageIoImpl();
    }

    public byte[] getImage(String imageFileLocation) {

        byte[] imageBytes = imageIo.getImage(imageFileLocation);
        // apply compression logic

        return imageBytes;
    }
}
```

---

## Spring data jpa parctical program github link:
https://github.com/Developers345/practical-spring-data-jpa-program.git

## Key Benefits of Proxy Design Pattern

* Avoids code duplication
* Follows **Single Responsibility Principle**
* Enables **runtime behavior extension**
* Clean separation of core logic and additional responsibilities


