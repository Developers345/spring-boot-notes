# Table per Concrete Class

## Overview
In the **Table per Concrete Class** strategy, for every concrete entity class within the inheritance hierarchy, a separate and independent database table is created.

- If the **parent entity class is non-abstract**, a separate database table is created to represent its attributes.
- For each **subclass**, a separate table is created containing:
  - All attributes of the parent entity class
  - All attributes of the subclass

This is why the strategy is called **Table per Concrete Class** — each concrete entity class in the inheritance hierarchy has its own table.

---

## Persistence Strategy Explanation

To perform persistence operations using this strategy, we must inform the Hibernate/JPA framework about the chosen inheritance mapping strategy.

### Important Guidelines
- **Avoid using the `IDENTITY` generator**
- Primary key values **must be unique across the entire inheritance hierarchy**

### Why Primary Key Uniqueness Matters
Example:
- If an `InsurancePlan` is stored with `plan_no = 1`
- We **must not** store an `AccidentalInsurancePlan` with `accidental_insurance_plan_no = 1`

Otherwise:
- Hibernate cannot support **polymorphic retrieval**
- Because the same identifier exists in multiple tables

When primary keys are unique across the hierarchy:
- Hibernate/JPA can quickly identify **which table** contains the record
- Correct entity object is returned
- **Polymorphic retrieval works properly**

---

## ID Generation Strategies

### Recommended
- `increment`
- `sequence`
- `uuid`
- `guid`

### Not Recommended
- `identity`

### Behavior of Generators

| Generator  | Description |
|-----------|-------------|
| identity  | ❌ Do not use |
| increment | Queries max(primary key) across all tables and adds `+1` |
| sequence  | Uses the **same sequence** across all entity tables |
| uuid      | Always unique |
| guid      | Always unique |
| select    | Hibernate only fetches the identifier |

---

## Increment Generator – Internal Query

```sql
select max(pno) from (  
  select plan_no 'pno' from insurance_plan
  union
  select accidental_insurance_plan_no 'pno' from accidental_insurance_plan
  union
  select automobile_insurance_plan_no 'pno' from automobile_insurance_plan
);
````

This ensures the generated ID is **unique across all tables**.

---

## Persistence Operations

### save(entityObject) / persist(entityObject)

#### Parent Entity

* Data is stored directly in the **parent table**
* Primary key is generated uniquely across the hierarchy

#### Child Entity

* Superclass attributes + subclass attributes are stored
* Data is inserted directly into the **subclass table**
* ID is generated uniquely across all tables

---

### get(classType, id) / fetch(classType, id)

#### Fetching a Child Entity

```java
session.get(AccidentalInsurancePlan.class, 2);
```

* Hibernate queries **only the subclass table**

#### Fetching a Superclass Entity (Polymorphic Retrieval)

* Hibernate queries **all tables** using `UNION`
* Identifies which table contains the record
* Returns the corresponding entity object

```sql
select ip.*, '' ic, '' dpc, '' fc, '' vt, 0 tno 
from insurance_plan ip where ip.plan_no = ?
union
select aip.*, '' fc, '' vt, 1 tno 
from accidental_insurance_plan aip where aip.accidental_insurance_plan_no = ?
union
select amip.*, '' ic, '' dpc, 2 tno 
from automobile_insurance_plan amip where amip.automobile_insurance_plan_no = ?
```

---

## Inheritance Mapping Strategy Comparison

| Strategy                  | Mechanism                           |
| ------------------------- | ----------------------------------- |
| Table per Class Hierarchy | Discriminator column (single table) |
| Table per Subclass        | Joined tables using FK              |
| Table per Concrete Class  | UNION, unique PK across hierarchy   |

---

## Hibernate Mapping Files

### InsurancePlan.hbm.xml

```xml
<hibernate-mapping package="com.tpcc.entities">
  <class name="InsurancePlan" table="insurance_plan">
    <id name="planNo" column="plan_no">
      <generator class="increment"/>
    </id>
    <property name="planName" column="plan_nm"/>
    <property name="minEligibilityAge" column="min_eligiblity_age"/>
    <property name="maxEligibilityAge" column="max_eligibility_age"/>
    <property name="minTenure" column="min_tenure"/>
    <property name="maxTenure" column="max_tenure"/>
    <property name="minAmount" column="min_amount"/>
  </class>
</hibernate-mapping>
```

---

### AccidentalInsurancePlan.hbm.xml

```xml
<hibernate-mapping package="com.tpcc.entities">
<!--Inheritance strategy is table per concreate class we are telling to hibernate through union-subclass tag-->
  <union-subclass 
      name="AccidentalInsurancePlan"
      table="accidental_insurance_plan"
      extends="InsurancePlan">
    <property name="internationalCoverage" column="international_coverage"/>
    <property name="disabilityCoveragePercentage" column="disability_coverage_percentage"/>
  </union-subclass>
</hibernate-mapping>
```

---

### AutomobileInsurancePlan.hbm.xml

```xml
<hibernate-mapping package="com.tpcc.entities">
  <union-subclass 
      name="AutomobileInsurancePlan"
      table="automobile_insurance_plan"
      extends="InsurancePlan">
    <property name="fullCoverage" column="full_coverage"/>
    <property name="vehicleType" column="vehicle_type"/>
  </union-subclass>
</hibernate-mapping>
```

---

## Entity Classes

### InsurancePlan.java

```java
package com.tpch.entities;
import java.io.Serializable;
import lombok.Data;

@Data
public class InsurancePlan implements Serializable {
  private static final long serialVersionUID = -1835824029528334250L;
  protected int planNo;
  protected String planName;
  protected int minEligibilityAge;
  protected int maxEligibilityAge;
  protected int minTenure;
  protected int maxTenure;
  protected double minAmount;
}
```

---

### AccidentalInsurancePlan.java

```java
package com.tpch.entities;
import lombok.Data;

@Data
public class AccidentalInsurancePlan extends InsurancePlan {
  protected boolean internationalCoverage;
  protected float disabilityCoveragePercentage;
}
```

---

### AutomobileInsurancePlan.java

```java
package com.tpch.entities;
import lombok.Data;

@Data
public class AutomobileInsurancePlan extends InsurancePlan {
  protected boolean fullCoverage;
  protected String vehicleType;
}
```

---

## SessionFactoryRegistry.java

```java
package com.tpch.helper;

import org.hibernate.SessionFactory;
import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;

public class SessionFactoryRegistry {
  private static SessionFactory sessionFactory =
      new MetadataSources(
          new StandardServiceRegistryBuilder().configure().build())
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
package com.tpch.dao;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import com.tpch.entities.InsurancePlan;
import com.tpch.helper.SessionFactoryRegistry;

public class InsurancePlanDao {

  public int saveInsurancePlan(InsurancePlan insurancePlan) {
    int planNo = 0;
    boolean flag = false;
    Session session = null;
    Transaction transaction = null;

    try {
      SessionFactory sessionFactory = SessionFactoryRegistry.getSessionFactory();
      session = sessionFactory.openSession();
      transaction = session.beginTransaction();

      planNo = (Integer) session.save(insurancePlan);
      flag = true;
    } finally {
      if (transaction != null) {
        if (flag) transaction.commit();
        else transaction.rollback();
      }
      if (session != null) session.close();
    }
    return planNo;
  }

  public InsurancePlan getInsurancePlan(Class<?> entityClass, int planNo) {
    Session session = null;
    InsurancePlan insurancePlan = null;

    try {
      SessionFactory sessionFactory = SessionFactoryRegistry.getSessionFactory();
      session = sessionFactory.openSession();
      insurancePlan = (InsurancePlan) session.get(entityClass, planNo);
    } finally {
      if (session != null) session.close();
    }
    return insurancePlan;
  }
}
```

---

## Hibernate Configuration

### hibernate.cfg.xml

```xml
<hibernate-configuration>
  <session-factory>
    <property name="connection.driver_class">com.mysql.cj.jdbc.Driver</property>
    <property name="connection.url">jdbc:mysql://localhost:3306/hdb</property>
    <property name="connection.username">root</property>
    <property name="connection.password">root</property>
    <property name="show_sql">true</property>
    <property name="hbm2ddl.auto">update</property>

    <mapping resource="InsurancePlan.hbm.xml"/>
    <mapping resource="AccidentalInsurancePlan.hbm.xml"/>
    <mapping resource="AutomobileInsurancePlan.hbm.xml"/>
  </session-factory>
</hibernate-configuration>
```

---

## Test Class

### TPCCTest.java

```java
package com.tpch.test;

import com.tpch.dao.InsurancePlanDao;
import com.tpch.entities.InsurancePlan;

public class TPCCTest {
  public static void main(String[] args) {
    InsurancePlanDao dao = new InsurancePlanDao();
    /* InsurancePlan ip = new InsurancePlan();
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

    InsurancePlan ip = dao.getInsurancePlan(InsurancePlan.class, 2);
    System.out.println(ip);
  }
}
```

---

## Maven Configuration

### pom.xml

```xml
<project>
  <modelVersion>4.0.0</modelVersion>

  <groupId>hibernate</groupId>
  <artifactId>tableperclasshierarchy</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <properties>
    <maven.compiler.source>17</maven.compiler.source>
    <maven.compiler.target>17</maven.compiler.target>
  </properties>

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
</project>
```
### output
==========

## Save Operation
-----------------

```sql
Hibernate: create table accidental_insurance_plan (
  plan_no integer not null,
  plan_nm varchar(255),
  min_eligibility_age integer,
  max_eligibility_age integer,
  min_tenure integer,
  max_tenure integer,
  min_amount float(53),
  international_coverage bit,
  disability_coverage_percentage float(23),
  primary key (plan_no)
) engine=InnoDB
````

```sql
Hibernate: create table automobile_insurance_plan (
  plan_no integer not null,
  plan_nm varchar(255),
  min_eligibility_age integer,
  max_eligibility_age integer,
  min_tenure integer,
  max_tenure integer,
  min_amount float(53),
  full_coverage bit,
  vehicle_type varchar(255),
  primary key (plan_no)
) engine=InnoDB
```

```sql
Hibernate: create table insurance_plan (
  plan_no integer not null,
  plan_nm varchar(255),
  min_eligibility_age integer,
  max_eligibility_age integer,
  min_tenure integer,
  max_tenure integer,
  min_amount float(53),
  primary key (plan_no)
) engine=InnoDB
```

### Increment ID Generator Query

```sql
Hibernate: select max(ids_.mx) from (
  select max(plan_no) as mx from accidental_insurance_plan
  union all
  select max(plan_no) as mx from automobile_insurance_plan
  union all
  select max(plan_no) as mx from insurance_plan
) ids_
```

### Insert Operations

```sql
Hibernate: insert into insurance_plan
(max_eligibility_age, max_tenure, min_amount, min_eligibility_age, min_tenure, plan_nm, plan_no)
values (?,?,?,?,?,?,?)
```

```
insurance plan no : 1
```

```sql
Hibernate: insert into accidental_insurance_plan
(max_eligibility_age, max_tenure, min_amount, min_eligibility_age, min_tenure, plan_nm,
 disability_coverage_percentage, international_coverage, plan_no)
values (?,?,?,?,?,?,?,?,?)
```

```
accidental insurance plan no : 2
```

```sql
Hibernate: insert into automobile_insurance_plan
(max_eligibility_age, max_tenure, min_amount, min_eligibility_age, min_tenure, plan_nm,
 full_coverage, vehicle_type, plan_no)
values (?,?,?,?,?,?,?,?,?)
```

```
automobile insurance plan no : 3
```

---

## Get (Retrieval Operation)

---

### Sub-class Retrieval

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
from accidental_insurance_plan aip1_0
where aip1_0.plan_no = ?
```

```
AccidentalInsurancePlan(
  internationalCoverage = true,
  disabilityCoveragePercentage = 50.0
)
```

---

### Super-class Retrieval (Polymorphic Access)

---

```sql
Hibernate: select
  ip1_0.plan_no,
  ip1_0.clazz_,
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
from (
  select
    plan_no,
    plan_nm,
    min_eligibility_age,
    max_eligibility_age,
    min_tenure,
    max_tenure,
    min_amount,
    null as international_coverage,
    null as disability_coverage_percentage,
    null as full_coverage,
    null as vehicle_type,
    0 as clazz_
  from insurance_plan

  union all

  select
    plan_no,
    plan_nm,
    min_eligibility_age,
    max_eligibility_age,
    min_tenure,
    max_tenure,
    min_amount,
    international_coverage,
    disability_coverage_percentage,
    null as full_coverage,
    null as vehicle_type,
    1 as clazz_
  from accidental_insurance_plan

  union all

  select
    plan_no,
    plan_nm,
    min_eligibility_age,
    max_eligibility_age,
    min_tenure,
    max_tenure,
    min_amount,
    null as international_coverage,
    null as disability_coverage_percentage,
    full_coverage,
    vehicle_type,
    2 as clazz_
  from automobile_insurance_plan
) ip1_0
where ip1_0.plan_no = ?
```

AutomobileInsurancePlan(
  fullCoverage = true,
  vehicleType = 4 Wheeler
)

### Pictorial Representation
<img width="3568" height="4172" alt="19-JUN-2021-TPCH" src="https://github.com/user-attachments/assets/5eb98959-330d-4b93-bb52-ff37fa4718c8" />
