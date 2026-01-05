# How to Write Custom Queries in Spring Data JPA

## Normal JPA Approach

### Example 1: Find Bike by Name
```java
EntityManagerFactory entityManagerFactory =
        EntityManagerFactoryHelper.getEntityManagerFactory();

EntityManager entityManager =
        entityManagerFactory.createEntityManager();

TypedQuery<Bike> typedQuery =
        entityManager.createQuery("from Bike where bikeName=?");

typedQuery.setParameter(1, "Honda");

List<Bike> bikes = typedQuery.getResultList();
````

---

### Example 2: Find Bikes by Price Range

```java
EntityManagerFactory entityManagerFactory =
        EntityManagerFactoryHelper.getEntityManagerFactory();

EntityManager entityManager =
        entityManagerFactory.createEntityManager();

TypedQuery<Bike> typedQuery =
        entityManager.createQuery("from Bike b where b.price between ? and ?");

typedQuery.setParameter(1, 10000);
typedQuery.setParameter(2, 40000);

List<Bike> bikes = typedQuery.getResultList();
```

---

## Problem with Normal JPA

* Only the **JPQL query string changes**.
* The remaining boilerplate code remains the same for every query.
* Leads to repetitive and verbose code.

---

## Spring Data JPA Solution

* Spring Data JPA allows writing **query methods using method names**.
* Internally, Spring Data JPA uses a **QueryBuilder** to derive JPQL queries from method names.

---

## Repository Example

```java
interface BikeRepository extends CrudRepository<Bike, Integer> {

    // Predefined CRUD methods are inherited from CrudRepository

    // Custom query method
    List<Bike> findByBikeName(String bikeName);

    // Based on method name, QueryBuilder derives the JPQL query
}
```

> Any additional methods defined in the repository interface are treated as **query methods**.

---

## Spring Data JPA Query Keywords

Spring Data JPA provides a rich set of keywords used by the query builder:

* `Distinct`
* `And`
* `Or`
* `Not`
* `Count`

  * Example: `countByProductName`
* `Is`, `Equals`

  * `findByProductName`
  * `findByProductNameIs`
  * `findByProductNameEquals(String productName)`
* `Between`

  * `findByPriceBetween(double minPrice, double maxPrice)`
* `In`

  * `findByProductNameIn(List<String> productNames)`
* `NotIn`
* `LessThan`
* `GreaterThan`
* `LessThanEqual`
* `GreaterThanEqual`
* `Like`
* `StartingWith`
* `OrderBy`

---

## Internal Working (Simplified)

```java
class QueryInvocationHandler implements InvocationHandler {

    private EntityManagerFactory entityManagerFactory;

    public Object invoke(Method method, Object[] args, Class<?> proxy) {

        EntityManager entityManager =
                entityManagerFactory.createEntityManager();

        String jpqlQuery =
                QueryBuilder.derive(method.getName());

        TypedQuery<Bike> typedQuery =
                entityManager.createQuery(jpqlQuery);

        typedQuery.setParameter(1, args[0]);
        typedQuery.setParameter(2, args[1]);

        return typedQuery.getResultList();
    }
}
```

---

## Spring Data JPA – Custom Query Variations

### 1. Derived Query (Method Name Based)

```java
public List<Bike> findByBikeName(String bikeName);
```

---

### 2. Positional Parameters

```java
@Query("from Bike b where b.bikeName=?1 and b.bikeModel=?2")
public Bike getByBikeNameModel(String bikeName, String bikeModel);
```

---

### 3. Named Parameters

```java
@Query("from Bike b where b.bikeName=:bikeName and b.bikeModel=:bikeModel and b.bikeColor=:bcolor")
public Bike getByBikeNameModelColor(
        String bikeName,
        String bikeModel,
        @Param("bcolor") String bikeColor
);
```

---

## Summary

* Normal JPA requires repetitive boilerplate code.
* Spring Data JPA:

  * Eliminates boilerplate
  * Supports query derivation from method names
  * Allows custom JPQL using `@Query`
* Internally relies on **runtime proxies** and **query derivation logic**.



```
```
