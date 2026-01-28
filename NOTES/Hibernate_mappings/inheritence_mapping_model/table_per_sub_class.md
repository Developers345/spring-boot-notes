# Inheritance Mapping in Hibernate (Table per Subclass)

There are **3 ways** we can map an inheritance relationship between entity classes into a relational model:

1. **Table per class hierarchy**
2. **Table per subclass**
3. **Table per concrete class**

---

## Note
- All inheritance mapping models support **1–to–1 relationship**.

---

## Problem with Table per Class Hierarchy

- If there are many subclasses in the hierarchy, we end up storing a large number of **NULL values** in the table.
- The data model is **not normalized**.
- Leads to **memory wastage**.

👉 To overcome the above problems, we go for **Table per Subclass**.

---

## Table per Subclass

- We create **one table for the parent entity class**.
- For **each subclass**, we create a **separate table**.
- Each subclass table has a **foreign key relationship** with the parent table.

### We need to provide two things to Hibernate/JPA:
1. Declare the **inheritance strategy**
2. Define the **relationship / foreign key column** of the subclass

Once configured, Hibernate/JPA can perform database operations correctly.

---

## Persistence Operations in Table per Subclass

### save(entityObject) / persist(entityObject)

- **Parent object**  
  → Stored directly into the **parent table**

- **Subclass object**  
  → Superclass attributes are stored in the **parent table**  
  → Generated **primary key** is reused as **foreign key** in the subclass table  
  → Subclass-specific attributes are stored in the subclass table

---

### get(entityObject) / fetch(entityObject)

#### Subclass object

Example:
```java
session.get(AccidentalInsurancePlan.class, 2);
````

Hibernate executes an **INNER JOIN**:

```sql
select ip.*, aip.*
from insurance_plan ip
inner join accidental_insurance_plan aip
on ip.plan_no = aip.accidental_insurance_plan_no
where ip.plan_no = 2;
```

**Note**

* Hibernate uses **INNER JOIN** for subclass retrieval.
* INNER JOIN fetches data only when records exist in **both tables**.

---

#### Parent class object

Example:

```java
session.get(InsurancePlan.class, 2);
```

Hibernate executes a **LEFT OUTER JOIN**:

```sql
select ip.*, aip.*, amip.*,
case
  when aip.plan_no is not null then '1'
  when amip.plan_no is not null then '2'
  else '0'
end
from insurance_plan ip
left outer join accidental_insurance_plan aip
on ip.plan_no = aip.accidental_insurance_plan_no
left outer join automobile_insurance_plan amip
on ip.plan_no = amip.automobile_insurance_plan_no
where ip.plan_no = 2;
```

**Note**

* Hibernate uses **LEFT OUTER JOIN** for superclass retrieval.

---

## Hibernate Mapping Files

### InsurancePlan.hbm.xml

```xml
<hibernate-mapping package="com.tpsc.entities">
  <class name="InsurancePlan" table="insurance_plan">
    <id name="planNo" column="plan_no">
      <generator class="increment"/>
    </id>
    <property name="planName" column="plan_nm"/>
    <property name="minEligibilityAge" column="min_eligibility_age"/>
    <property name="maxElibilityAge" column="max_eligibility_age"/>
    <property name="minTenure" column="min_tenure"/>
    <property name="maxTenure" column="max_tenure"/>
    <property name="minAmount" column="min_amount"/>
  </class>
</hibernate-mapping>
```

---

### AccidentalInsurancePlan.hbm.xml

```xml
<hibernate-mapping package="com.tpsc.entities">
<!-- Inheritance Strategy -->
  <joined-subclass
      name="AccidentalInsurancePlan"
      table="accidental_insurance_plan"
      extends="InsurancePlan">
    <key column="accidental_insurance_plan_no"/>
    <!--This column is in relationship with key column of parent table -->
    <property name="internationalCoverage" column="international_coverage"/>
    <property name="disabilityCoveragePercentage"
              column="disablity_coverage_percentage"/>
  </joined-subclass>
</hibernate-mapping>
```

---

### AutomobileInsurancePlan.hbm.xml

```xml
<hibernate-mapping package="com.tpsc.entities">
  <joined-subclass
      name="AutomobileInsurancePlan"
      table="automobile_insurance_plan"
      extends="InsurancePlan">
    <key column="automobile_insurance_plan_no"/>
    <property name="fullCoverage" column="full_coverage"/>
    <property name="vehicleType" column="vehicle_type"/>
  </joined-subclass>
</hibernate-mapping>
```

---

## Full Example – Table per Subclass

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

### SessionFactoryRegistry.java

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
      SessionFactory sessionFactory =
          SessionFactoryRegistry.getSessionFactory();
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
      SessionFactory sessionFactory =
          SessionFactoryRegistry.getSessionFactory();
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

### hibernate.cfg.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
  "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
  "https://hibernate.org/dtd/hibernate-configuration-3.0.dtd">
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

### TPSCTest.java

```java
package com.tpch.test;

import com.tpch.dao.InsurancePlanDao;
import com.tpch.entities.InsurancePlan;

public class TPSCTest {
  public static void main(String[] args) {
    InsurancePlanDao dao = new InsurancePlanDao();

    InsurancePlan ip =
        dao.getInsurancePlan(InsurancePlan.class, 2);
    System.out.println(ip);
  }
}
```

## Output

---

### Save Operation

```text
Hibernate: create table accidental_insurance_plan (
  accidental_insurance_plan_no integer not null,
  international_coverage bit,
  disability_coverage_percentage float(23),
  primary key (accidental_insurance_plan_no)
) engine=InnoDB

Hibernate: create table automobile_insurance_plan (
  automobile_insurance_plan_no integer not null,
  full_coverage bit,
  vehicle_type varchar(255),
  primary key (automobile_insurance_plan_no)
) engine=InnoDB

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

Hibernate: alter table accidental_insurance_plan
add constraint FKnhncsja9e4rpq7yaosdammpio
foreign key (accidental_insurance_plan_no)
references insurance_plan (plan_no)

Hibernate: alter table automobile_insurance_plan
add constraint FK6jq6ithahn549yv7y08724k8o
foreign key (automobile_insurance_plan_no)
references insurance_plan (plan_no)

Hibernate: select max(plan_no) from insurance_plan

Hibernate: insert into insurance_plan
(max_eligibility_age, max_tenure, min_amount,
 min_eligibility_age, min_tenure, plan_nm, plan_no)
values (?,?,?,?,?,?,?)
insurance plan no : 1

Hibernate: insert into insurance_plan
(max_eligibility_age, max_tenure, min_amount,
 min_eligibility_age, min_tenure, plan_nm, plan_no)
values (?,?,?,?,?,?,?)

Hibernate: insert into accidental_insurance_plan
(disability_coverage_percentage, international_coverage,
 accidental_insurance_plan_no)
values (?,?,?)
accidental insurance plan no : 2

Hibernate: insert into insurance_plan
(max_eligibility_age, max_tenure, min_amount,
 min_eligibility_age, min_tenure, plan_nm, plan_no)
values (?,?,?,?,?,?,?)

Hibernate: insert into automobile_insurance_plan
(full_coverage, vehicle_type, automobile_insurance_plan_no)
values (?,?,?)
automobile insurance plan no : 3
````

---

### Get Operation

#### Sub-class

```text
Hibernate: select
  aip1_0.accidental_insurance_plan_no,
  aip1_1.max_eligibility_age,
  aip1_1.max_tenure,
  aip1_1.min_amount,
  aip1_1.min_eligibility_age,
  aip1_1.min_tenure,
  aip1_1.plan_nm,
  aip1_0.disability_coverage_percentage,
  aip1_0.international_coverage
from accidental_insurance_plan aip1_0
join insurance_plan aip1_1
on aip1_0.accidental_insurance_plan_no = aip1_1.plan_no
where aip1_0.accidental_insurance_plan_no = ?

AccidentalInsurancePlan(
  internationalCoverage=true,
  disabilityCoveragePercentage=50.0
)
```

---

#### Super-class

```text
Hibernate: select
  ip1_0.plan_no,
  case
    when ip1_1.accidental_insurance_plan_no is not null then 1
    when ip1_2.automobile_insurance_plan_no is not null then 2
    when ip1_0.plan_no is not null then 0
  end,
  ip1_0.max_eligibility_age,
  ip1_0.max_tenure,
  ip1_0.min_amount,
  ip1_0.min_eligibility_age,
  ip1_0.min_tenure,
  ip1_0.plan_nm,
  ip1_1.disability_coverage_percentage,
  ip1_1.international_coverage,
  ip1_2.full_coverage,
  ip1_2.vehicle_type
from insurance_plan ip1_0
left join accidental_insurance_plan ip1_1
  on ip1_0.plan_no = ip1_1.accidental_insurance_plan_no
left join automobile_insurance_plan ip1_2
  on ip1_0.plan_no = ip1_2.automobile_insurance_plan_no
where ip1_0.plan_no = ?

AutomobileInsurancePlan(
  fullCoverage=true,
  vehicleType=4 Wheeler
)
```
#### Pictorial Representation - Table Per Sub-Class
<img width="3568" height="4172" alt="17-JUN-2021-TPSC" src="https://github.com/user-attachments/assets/000e9a78-efbc-4685-9745-5b66826ec8b0" />
