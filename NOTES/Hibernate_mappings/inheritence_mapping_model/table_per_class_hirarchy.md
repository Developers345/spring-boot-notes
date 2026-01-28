# Entity Relationship Mapping

## Overview
**Entity Relationship Mapping** explains how to represent relationships between entity classes and their corresponding table relationships in a relational database.

There are **two ways** classes can be related:
1. **Inheritance**
2. **Association**

From these, we derive **two types of Entity Relationship Mappings**:
1. **Inheritance Mapping Model**
2. **Association Mapping Model**

---

## Inheritance Mapping Model
This model helps map **inheritance relationships** between entity classes into a relational model.

There are **three strategies** to map inheritance:
1. **Table per Class Hierarchy**
2. **Table per Subclass**
3. **Table per Concrete Class**

Hibernate/JPA must be informed about which strategy is chosen so it can correctly handle persistence operations.

---

## Table per Class Hierarchy

### Concept
- All entity classes in the inheritance hierarchy are mapped to **one single table**.
- All attributes from all classes become columns in this table.

### Discriminator Column
Since multiple entity types share one table:
- We introduce a **discriminator column**
- This column stores a **discriminator value** that identifies which class a record belongs to.

### Mapping Requirements
To support this strategy, Hibernate/JPA must know:
- All entities map to **one table**
- Subclasses extend a **common superclass**
- A **discriminator column** exists
- Each class has a **unique discriminator value**

---

## Hibernate Behavior

### Save / Persist Operation

#### Superclass
- Stores superclass attributes
- Inserts discriminator value for the superclass

#### Subclass
- Stores subclass attributes
- Also stores inherited superclass attributes
- Inserts subclass-specific discriminator value

---

### Get Operation (`get(Class, id)`)

#### Subclass Retrieval
- Hibernate checks:
  - `id`
  - `discriminator-column`
- If discriminator value does not match, Hibernate returns `null`

Example:
```java
AccidentalInsurancePlan plan =
    session.get(AccidentalInsurancePlan.class, 3);
````

If `plan_no = 3` belongs to `AutomobileInsurancePlan`, Hibernate will **not** return an object.

Generated SQL:

```sql
SELECT * 
FROM insurance_plan 
WHERE plan_type = ? AND plan_no = ?
```

---

#### Superclass Retrieval (Polymorphic Access)

* Hibernate queries using only the primary key
* Then inspects the discriminator value
* Instantiates the appropriate subclass

Example:

```java
InsurancePlan ip =
    session.get(InsurancePlan.class, 2);
```

Even though the requested type is `InsurancePlan`, Hibernate returns the actual subclass object.

Generated SQL:

```sql
SELECT *
FROM insurance_plan
WHERE plan_no = ?
```

---

## Hibernate Mapping Files

### InsurancePlan.hbm.xml

```xml
<hibernate-mapping package="com.ihmm.entities">
  <class name="InsurancePlan" table="insurance_plan"
         discriminator-value="ip">
    <id name="planNo" column="plan_no">
      <generator class="increment"/>
    </id>
    <discriminator column="plan_type" type="string"/>
    <property name="planName" column="plan_nm"/>
    <property name="minEligibilityAge" column="min_eligibility_age"/>
    <property name="maxEligibilityAge" column="max_eligibility_age"/>
    <property name="minTenure" column="min_tenure"/>
    <property name="maxTenure" column="max_tenure"/>
    <property name="minAmount" column="main_amount"/>
  </class>
</hibernate-mapping>
```

---

### AccidentalInsurancePlan.hbm.xml

```xml
<hibernate-mapping package="com.ihmm.entities">
  <subclass name="AccidentalInsurancePlan"
            extends="InsurancePlan"
            discriminator-value="aip">
    <property name="internationalCoverage"
              column="international_coverage"/>
    <property name="disabilityPercentageCoverage"
              column="disability_percentage_coverage"/>
  </subclass>
</hibernate-mapping>
```

---

### AutomobileInsurancePlan.hbm.xml

```xml
<hibernate-mapping package="com.ihmm.entities">
  <subclass name="AutomobileInsurancePlan"
            extends="InsurancePlan"
            discriminator-value="amip">
    <property name="vehicleType" column="vehicle_type"/>
    <property name="fullCoverage" column="full_coverage"/>
  </subclass>
</hibernate-mapping>
```

---

## Full Example: Table per Class Hierarchy

### Entity Classes

#### InsurancePlan.java

```java
@Data
public class InsurancePlan implements Serializable {
    protected int planNo;
    protected String planName;
    protected int minEligibilityAge;
    protected int maxEligibilityAge;
    protected int minTenure;
    protected int maxTenure;
    protected double minAmount;
}
```

#### AccidentalInsurancePlan.java

```java
@Data
public class AccidentalInsurancePlan extends InsurancePlan {
    protected boolean internationalCoverage;
    protected float disabilityCoveragePercentage;
}
```

#### AutomobileInsurancePlan.java

```java
@Data
public class AutomobileInsurancePlan extends InsurancePlan {
    protected boolean fullCoverage;
    protected String vehicleType;
}
```

---

## SessionFactory Utility

### SessionFactoryRegistry.java

```java
public class SessionFactoryRegistry {
    private static SessionFactory sessionFactory =
        new MetadataSources(
            new StandardServiceRegistryBuilder()
                .configure()
                .build())
            .buildMetadata()
            .getSessionFactoryBuilder()
            .applyAutoClosing(true)
            .build();

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
}
```

---

## DAO Layer

### InsurancePlanDao.java

```java
public class InsurancePlanDao {

    public int saveInsurancePlan(InsurancePlan insurancePlan) {
        Session session = null;
        Transaction tx = null;
        int planNo = 0;

        try {
            session = SessionFactoryRegistry
                        .getSessionFactory()
                        .openSession();
            tx = session.beginTransaction();
            planNo = (Integer) session.save(insurancePlan);
            tx.commit();
        } finally {
            if (session != null) session.close();
        }
        return planNo;
    }

    public InsurancePlan getInsurancePlan(
            Class<?> entityClass, int planNo) {
        Session session = null;
        InsurancePlan plan = null;
        try {
            session = SessionFactoryRegistry
                        .getSessionFactory()
                        .openSession();
            plan = (InsurancePlan)
                   session.get(entityClass, planNo);
        } finally {
            if (session != null) session.close();
        }
        return plan;
    }
}
```

---

## Hibernate Configuration

### hibernate.cfg.xml

```xml
<hibernate-configuration>
  <session-factory>
    <property name="connection.driver_class">
        com.mysql.cj.jdbc.Driver
    </property>
    <property name="connection.url">
        jdbc:mysql://localhost:3306/hdb
    </property>
    <property name="connection.username">root</property>
    <property name="connection.password">root</property>
    <property name="hibernate.show_sql">true</property>
    <property name="hibernate.hbm2ddl.auto">update</property>

    <mapping resource="InsurancePlan.hbm.xml"/>
    <mapping resource="AccidentalInsurancePlan.hbm.xml"/>
    <mapping resource="AutomobileInsurancePlan.hbm.xml"/>
  </session-factory>
</hibernate-configuration>
```

---

## Test Class

### TPCHTest.java

```java
public class TPCHTest {
    public static void main(String[] args) {
        InsurancePlanDao dao = new InsurancePlanDao();
        /*InsurancePlan ip = new InsurancePlan();
		  ip.setPlanName("Jeevan bheema");
		  ip.setMinEligibilityAge(18);
		  ip.setMaxEligibilityAge(60);
		  ip.setMinTenure(5);
		  ip.setMaxTenure(40);
		  ip.setMinAmount(500000);

		  int ipPlanNo = dao.saveInsurancePlan(ip);
		  System.out.println("insurance plan no : " + ipPlanNo);

		  AccidentalInsurancePlan aip = new AccidentalInsurancePlan();
		  aip.setPlanName("Jeevan suraksha");
		  aip.setMinEligibilityAge(21);
		  aip.setMaxEligibilityAge(59);
		  aip.setMinTenure(1);
		  aip.setMaxTenure(3);
		  aip.setMinAmount(100000);
		  aip.setInternationalCoverage(true);
		  aip.setDisabilityCoveragePercentage(50);
		  int aipPlanNo =
		  dao.saveInsurancePlan(aip);

		  System.out.println("accidental insurance plan no : " + aipPlanNo);

		  AutomobileInsurancePlan amip = new AutomobileInsurancePlan();
		  amip.setPlanName("totalprotection");
		  amip.setMinEligibilityAge(0);
		  amip.setMaxEligibilityAge(16);
		  amip.setMinTenure(1); amip.setMaxTenure(3);
		  amip.setMinAmount(100000);
		  amip.setFullCoverage(true);
		  amip.setVehicleType("4 Wheeler");

		  int amipPlanNo = dao.saveInsurancePlan(amip);
		 System.out.println("automobile insurance plan no : " + amipPlanNo);*/

        InsurancePlan ip =
            dao.getInsurancePlan(InsurancePlan.class, 2);
        System.out.println(ip);
    }
}
```

---

## Maven Configuration

### pom.xml

```xml
<dependencies>
  <dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>6.4.4.Final</version>
  </dependency>

  <dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.2.0</version>
  </dependency>

  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
    <scope>provided</scope>
  </dependency>
</dependencies>
```

---

## Output

### Save Operation

* Single table created
* Discriminator column populated automatically

### Retrieval Operations

#### Subclass Retrieval

```sql
SELECT ... FROM insurance_plan
WHERE plan_type='aip' AND plan_no=?
```

#### Superclass (Polymorphic) Retrieval

```sql
SELECT ... FROM insurance_plan
WHERE plan_no=?
```

Hibernate returns the correct subclass instance based on the discriminator value.
               
               (or)
## Output

---

## Save Operation
----------------

```sql
Hibernate: create table insurance_plan (
  plan_no integer not null auto_increment,
  plan_type varchar(4) not null,
  plan_nm varchar(255),
  min_eligibility_age integer,
  max_eligibility_age integer,
  min_tenure integer,
  max_tenure integer,
  min_amount float(53),
  international_coverage bit,
  disability_coverage_percentage float(23),
  full_coverage bit,
  vehicle_type varchar(255),
  primary key (plan_no)
) engine=InnoDB
````

```sql
Hibernate: insert into insurance_plan
(max_eligibility_age, max_tenure, min_amount, min_eligibility_age,
 min_tenure, plan_nm, plan_type)
values (?,?,?,?,?,?,'ip')
```

```
insurance plan no : 1
```

```sql
Hibernate: insert into insurance_plan
(max_eligibility_age, max_tenure, min_amount, min_eligibility_age,
 min_tenure, plan_nm, disability_coverage_percentage,
 international_coverage, plan_type)
values (?,?,?,?,?,?,?,?,'aip')
```

```
accidental insurance plan no : 2
```

```sql
Hibernate: insert into insurance_plan
(max_eligibility_age, max_tenure, min_amount, min_eligibility_age,
 min_tenure, plan_nm, full_coverage, vehicle_type, plan_type)
values (?,?,?,?,?,?,?,?,'amip')
```

```
automobile insurance plan no : 3
```

---

## Retrieval Operation (`get`)

---

### AccidentalInsurancePlan (Sub-class)

---

```sql
Hibernate: select
  aip1_0.plan_no,
  aip1_0.max_eligibility_age,
  aip1_0.max_tenure,
  aip1_0.min_amount,
  aip1_0.min_eligibility_age,
  aip1_0.min_tenure,
  aip1_0.plan_nm,
  aip1_0.disability_coverage_percentage,
  aip1_0.international_coverage
from insurance_plan aip1_0
where aip1_0.plan_type='aip'
  and aip1_0.plan_no=?
```

```
AccidentalInsurancePlan(
  internationalCoverage=true,
  disabilityCoveragePercentage=50.0
)
```

---

### AutomobileInsurancePlan (Sub-class)

---

```sql
Hibernate: select
  aip1_0.plan_no,
  aip1_0.max_eligibility_age,
  aip1_0.max_tenure,
  aip1_0.min_amount,
  aip1_0.min_eligibility_age,
  aip1_0.min_tenure,
  aip1_0.plan_nm,
  aip1_0.full_coverage,
  aip1_0.vehicle_type
from insurance_plan aip1_0
where aip1_0.plan_type='amip'
  and aip1_0.plan_no=?
```

```
AutomobileInsurancePlan(
  fullCoverage=true,
  vehicleType=4 Wheeler
)
```

---

### InsurancePlan (Super-class – Polymorphic Retrieval)

---

```sql
Hibernate: select
  ip1_0.plan_no,
  ip1_0.plan_type,
  ip1_0.max_eligibility_age,
  ip1_0.max_tenure,
  ip1_0.min_amount,
  ip1_0.min_eligibility_age,
  ip1_0.min_tenure,
  ip1_0.plan_nm,
  ip1_0.disability_coverage_percentage,
  ip1_0.international_coverage,
  ip1_0.full_coverage,
  ip1_0.vehicle_type
from insurance_plan ip1_0
where ip1_0.plan_no=?
```

```
AutomobileInsurancePlan(
  fullCoverage=true,
  vehicleType=4 Wheeler
)
```
