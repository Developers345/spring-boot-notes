# JPA API
-------

- Every database table can be represented by **one Entity class** that reflects the structure of the table.

- **Hibernate / JPA** needs to connect to the database using `persistence.xml`.  
  In `persistence.xml`, a **persistence-unit** represents **one database** along with its related entity configuration.

- In JPA, **EntityManager** performs the database operations.

---

## EntityManager and EntityManagerFactory

- `EntityManager` needs mapping information and database configuration from `persistence.xml`.  
  Reading this information is a **complex and costly operation**.

- Object creation is complex, so `EntityManager` delegates this responsibility to **EntityManagerFactory**.

- `EntityManagerFactory` itself does not directly read `persistence.xml`.  
  It takes help from **PersistenceContext**.

- **PersistenceContext**:
  - Reads `persistence.xml`
  - Reads entity mapping information
  - Loads data from `META-INF/persistence.xml` (classpath)
  - Passes this information to `EntityManagerFactory`

- `EntityManagerFactory` holds:
  - Database information
  - Mapping metadata for entities

---

## Creating EntityManager

```java
// EntityManagerFactory is an interface
EntityManagerFactory emf = PersistenceContext.createEntityManagerFactory("pu");

// EntityManager is an interface
EntityManager em = emf.createEntityManager();
````

---

## Number of EntityManagerFactory Instances

* **One EntityManagerFactory per database**
* If an application works with **2 databases**, then **2 EntityManagerFactory instances** are created.

---

## Responsibilities NOT Handled by EntityManager

`EntityManager` does **NOT** handle the following because they are costly and inefficient to repeat for every operation:

* Reading `persistence.xml` to identify the database
* Opening and closing database connections repeatedly
* Reading entity mapping information from annotations
  (These are I/O operations and expensive)

---

## Responsibilities Handled by EntityManager

* Performing database operations such as:

```java
EntityManager.persist(bike);
```

* Internally:

  * Creates prepared statements
  * Executes `executeUpdate()` / `executeQuery()`

---

## Responsibilities of EntityManagerFactory

`EntityManagerFactory` takes care of:

* Holding JPA configuration and mapping metadata **in memory**
* Acting as a **factory** for creating `EntityManager` objects
* Managing the **connection pool**
* Providing database connections to `EntityManager`

# Example Program
----------------

## BikeEntity.java
-----------------

```java
@Entity
@Table(name = "bike")
public class BikeBo {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "bike_no")
    private int bikeNo;

    @Column(name = "bike_name")
    private String bikeName;

    @Column(name = "bike_color")
    private String bikeColor;

    @Column(name = "bike_model")
    private String bikeModel;

    // no-arg constructor
    // accessor (getter/setter) methods
}
````

---

## JPA Supported ID Generators

JPA supports the following ID generation strategies:

1. `AUTO`
2. `IDENTITY`
3. `SEQUENCE`
4. `TABLE`

---

## META-INF/persistence.xml

---

```xml
<?xml version="1.0" encoding="utf-8"?>
<persistence>
    <persistence-unit name="rapidoPU">
        <provider>HibernateJpaPersistenceProvider</provider>
        <class>com.rapido.entites.Bike</class>

        <properties>
            <property name="javax.persistence.jdbc.driver" value="com.mysql.cj.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/rapido"/>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="root"/>

            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.dialect" value=""/>
        </properties>
    </persistence-unit>
</persistence>
```

---

## EntityManagerFactoryHelper.java

---

```java
class EntityManagerHepler {

    private static EntityManagerFactory entityManagerFactory;

    static {
        entityManagerFactory =
                PersistenceContext.createEntityManagerFactory("rapidoPU");
    }

    public static EntityManagerFactory getEntityManagerFactory() {
        return entityManagerFactory;
    }
}
```

---

## Test.java

---

```java
public class Test {

    public static void main(String[] args) {

        EntityManagerFactory entityManagerFactory = null;
        EntityManager entityManager = null;
        EntityTransaction entityTransaction = null;
        Bike bike = null;
        boolean flag = false;

        bike = new Bike();
        // populate the data into bike object

        try {
            entityManagerFactory =
                    EntityManagerFactoryHepler.getEntityManagerFactory();

            entityManager = entityManagerFactory.createEntityManager();
            entityTransaction = entityManager.getTransaction();

            entityTransaction.begin();
            entityManager.persist(bike);
            flag = true;

        } finally {

            if (flag) {
                entityTransaction.commit();
            } else {
                entityTransaction.rollback();
            }
        }

        entityManager.close();
        entityManagerFactory.close();
    }
}
```

---

## Problems with JPA API (Boilerplate Code)

---

While working with the JPA API, there is a lot of boilerplate code involved. Some of the major problems are:

1. Writing code to instantiate `EntityManagerFactory`.
2. Ensuring **only one `EntityManagerFactory` per database**, often by writing helper classes.
3. Creating `EntityManager` and closing it after use.
4. Closing `EntityManagerFactory` at the end of the application.
5. Writing explicit code to manage **transactions**.

