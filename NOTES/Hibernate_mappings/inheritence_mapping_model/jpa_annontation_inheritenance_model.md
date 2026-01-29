# How to Work with Inheritance Mapping in JPA API?

JPA API supports **3 strategies of inheritance mapping**:

1. **SINGLE_TABLE** = Table per class hierarchy (discriminator column)
2. **JOINED** = Table per subclass (foreign key column)
3. **TABLE_PER_CLASS** = Table per concrete class (no discriminator, no join)

---

## 1. SINGLE_TABLE Strategy
(Table Per Class Hierarchy)

In this strategy, **all entity classes in the inheritance hierarchy are mapped to a single table**.  
A discriminator column is used to differentiate between entity types.

### Example

```java
@Entity
@Table(name = "insurance_plan")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "plan_type")
@DiscriminatorValue("ip")
class InsurancePlan {

    @Id
    @Column(name = "plan_no")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    int planNo;

    @Column(name = "plan_nm")
    String planName;

    @Column(name = "min_eligibility_age")
    int minEligibilityAge;

    @Column(name = "max_elgibility_age")
    int maxEligibilityAge;

    @Column(name = "min_tenure")
    int minTenure;

    @Column(name = "max_tenure")
    int maxTenure;

    @Column(name = "min_amount")
    double minAmount;

    // accessors
}
````

```java
@Entity
@DiscriminatorValue("aip")
class AccidentalInsurancePlan extends InsurancePlan {

    @Column(name = "international_coverage")
    boolean internationalCoverage;

    @Column(name = "disability_coverage_percentage")
    double disabilityCoveragePercentage;
}
```

```java
@Entity
@DiscriminatorValue("amip")
class AutomobileInsurancePlan extends InsurancePlan {

    @Column(name = "full_coverage")
    boolean fullCoverage;

    @Column(name = "vehicle_type")
    String vehicleType;
}
```

---

## 2. JOINED Strategy

(Table Per Subclass)

In this strategy, **a separate table is created for each subclass**, and each subclass table is joined with the superclass table using a foreign key.

### Example

```java
@Entity
@Table(name = "insurance_plan")
@Inheritance(strategy = InheritanceType.JOINED)
class InsurancePlan {

    @Id
    @Column(name = "plan_no")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    int planNo;

    @Column(name = "plan_nm")
    String planName;

    @Column(name = "min_eligibility_age")
    int minEligibilityAge;

    @Column(name = "max_elgibility_age")
    int maxEligibilityAge;

    @Column(name = "min_tenure")
    int minTenure;

    @Column(name = "max_tenure")
    int maxTenure;

    @Column(name = "min_amount")
    double minAmount;

    // accessors
}
```

```java
@Entity
@PrimaryKeyJoinColumn
class AccidentalInsurancePlan extends InsurancePlan {

    @Column(name = "international_coverage")
    boolean internationalCoverage;

    @Column(name = "disability_coverage_percentage")
    double disabilityCoveragePercentage;
}
```

```java
@Entity
@PrimaryKeyJoinColumn
class AutomobileInsurancePlan extends InsurancePlan {

    @Column(name = "full_coverage")
    boolean fullCoverage;

    @Column(name = "vehicle_type")
    String vehicleType;
}
```

---

## 3. TABLE_PER_CLASS Strategy

(Table Per Concrete Class)

In this strategy, **each concrete entity class has its own table**.
There is **no discriminator column and no join** between tables.

### Example

```java
@Entity
@Table(name = "insurance_plan")
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
class InsurancePlan {

    @Id
    @Column(name = "plan_no")
    @GeneratedValue(strategy = GenerationType.TABLE)
    int planNo;

    @Column(name = "plan_nm")
    String planName;

    @Column(name = "min_eligibility_age")
    int minEligibilityAge;

    @Column(name = "max_elgibility_age")
    int maxEligibilityAge;

    @Column(name = "min_tenure")
    int minTenure;

    @Column(name = "max_tenure")
    int maxTenure;

    @Column(name = "min_amount")
    double minAmount;

    // accessors
}
```

```java
@Entity
class AccidentalInsurancePlan extends InsurancePlan {

    @Column(name = "international_coverage")
    boolean internationalCoverage;

    @Column(name = "disability_coverage_percentage")
    double disabilityCoveragePercentage;
}
```

```java
@Entity
class AutomobileInsurancePlan extends InsurancePlan {

    @Column(name = "full_coverage")
    boolean fullCoverage;

    @Column(name = "vehicle_type")
    String vehicleType;
}
```

---

## Inheritance Mapping Strategies Supported by JPA API

There are **3 inheritance strategies supported by JPA API**:

1. **SINGLE_TABLE**
2. **JOINED**
3. **TABLE_PER_CLASS**

---

### 1. SINGLE_TABLE

This strategy is the same as **Table Per Class Hierarchy**.
All entity classes in the inheritance hierarchy share **one single table**.

```java
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "columnName")
@DiscriminatorValue("value")
```

---

### 2. JOINED

This strategy is equivalent to **Table Per Subclass**.
Each subclass has its own table and is joined with the superclass table.

```java
@Inheritance(strategy = InheritanceType.JOINED)
```

For each subclass, use:

```java
@PrimaryKeyJoinColumn
```

This indicates that the subclass primary key is also a foreign key referencing the superclass primary key.

---

### 3. TABLE_PER_CLASS

This strategy is equivalent to **Table Per Concrete Class**.
Each entity class has its **own independent table**.

```java
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
```

---

## Example Domain Model

### Tour (Base Class)

* tourNo
* tourPackageName
* source
* destination
* days
* amount

### Domestic Tour (Subclass)

* additionalStateCharges
* tollTaxesTobePaid

### International Tour (Subclass)

* passportNo
* visaRequired
* visaCharges
