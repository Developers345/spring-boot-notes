# Full Example – Annotation SINGLE_TABLE (Table Per Class Hierarchy)
-------------------------------------------------------------------

## Tour.java
```java
package com.annonsingletable.entities;

import java.io.Serializable;

import jakarta.persistence.Column;
import jakarta.persistence.DiscriminatorColumn;
import jakarta.persistence.DiscriminatorValue;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Inheritance;
import jakarta.persistence.InheritanceType;
import jakarta.persistence.Table;

import lombok.Data;

@Data
@Entity
@Table(name = "tour")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tour_type")
@DiscriminatorValue("local")
public class Tour implements Serializable {

    @Id
    @Column(name = "tour_no")
    @GeneratedValue(strategy = GenerationType.TABLE)
    protected int tourNo;

    @Column(name = "tour_package_nm")
    protected String tourPackageName;

    protected int days;
    protected String source;
    protected String destination;
    protected double amount;
}
````

---

## DomesticTour.java

```java
package com.annonsingletable.entities;

import jakarta.persistence.Column;
import jakarta.persistence.DiscriminatorValue;
import jakarta.persistence.Entity;

import lombok.Data;
import lombok.ToString;

@Data
@ToString(callSuper = true)
@Entity
@DiscriminatorValue("domestic")
public class DomesticTour extends Tour {

    @Column(name = "additional_state_charges")
    protected double additionalStateCharges;

    @Column(name = "toll_taxes_to_be_paid")
    protected double tollTaxesToBePaid;
}
```

---

## InternationalTour.java

```java
package com.annonsingletable.entities;

import jakarta.persistence.Column;
import jakarta.persistence.DiscriminatorValue;
import jakarta.persistence.Entity;

import lombok.Data;
import lombok.ToString;

@Data
@ToString(callSuper = true)
@Entity
@DiscriminatorValue("international")
public class InternationalTour extends Tour {

    @Column(name = "passport_no")
    protected String passportNo;

    @Column(name = "visa_required")
    protected boolean visaRequired;

    @Column(name = "visa_charges")
    protected double visaCharges;
}
```

---

## persistence.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.1"
    xmlns="http://xmlns.jcp.org/xml/ns/persistence"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://xmlns.jcp.org/xml/ns/persistence
        http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd">

    <persistence-unit name="hdbunit">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <class>com.annonsingletable.entities.Tour</class>
        <class>com.annonsingletable.entities.DomesticTour</class>
        <class>com.annonsingletable.entities.InternationalTour</class>

        <properties>
            <property name="javax.persistence.jdbc.driver"
                      value="com.mysql.cj.jdbc.Driver" />
            <property name="javax.persistence.jdbc.url"
                      value="jdbc:mysql://localhost:3306/hdb" />
            <property name="javax.persistence.jdbc.user" value="root" />
            <property name="javax.persistence.jdbc.password" value="root" />

            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.hbm2ddl.auto" value="update" />
        </properties>
    </persistence-unit>
</persistence>
```

---

## EntityManagerFactoryRegistry.java

```java
package com.annonsingletable.helper;

import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.Persistence;

public class EntityManagerFactoryRegistry {

    private static EntityManagerFactory emf =
        Persistence.createEntityManagerFactory("hdbunit");

    public static EntityManagerFactory getEntityManagerFactory() {
        return emf;
    }

    public static void closeEntityManagerFactory() {
        if (emf != null) {
            emf.close();
            emf = null;
        }
    }
}
```

---

## TourDao.java

```java
package com.annonsingletable.dao;

import com.annonsingletable.entities.Tour;
import com.annonsingletable.helper.EntityManagerFactoryRegistry;

import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.EntityTransaction;

public class TourDao {

    public int saveTour(Tour tour) {
        EntityTransaction transaction = null;
        EntityManagerFactory emf = null;
        EntityManager em = null;
        boolean flag = false;
        int tourNo = 0;

        try {
            emf = EntityManagerFactoryRegistry.getEntityManagerFactory();
            em = emf.createEntityManager();
            transaction = em.getTransaction();
            transaction.begin();

            em.persist(tour);
            tourNo = tour.getTourNo();
            flag = true;
        } finally {
            if (transaction != null) {
                if (flag) transaction.commit();
                else transaction.rollback();
                em.close();
            }
        }
        return tourNo;
    }

    public <S extends Tour> Tour find(Class<S> clazzType, Object id) {
        EntityManagerFactory emf = null;
        EntityManager em = null;
        Tour tour = null;

        try {
            emf = EntityManagerFactoryRegistry.getEntityManagerFactory();
            em = emf.createEntityManager();
            tour = em.find(clazzType, id);
        } finally {
            if (em != null) em.close();
        }
        return tour;
    }
}
```

---

## AnnonSingleTableTest.java

```java
package com.annonsingletable.test;

import com.annonsingletable.dao.TourDao;
import com.annonsingletable.entities.DomesticTour;
import com.annonsingletable.entities.InternationalTour;
import com.annonsingletable.entities.Tour;

public class AnnonSingleTableTest {

    public static void main(String[] args) {

        TourDao dao = new TourDao();
        tour = new Tour();
		  tour.setSource("Hyderabad");
		  tour.setDestination("Warangal");
		  tour.setTourPackageName("local state tour for historic");
		  tour.setDays(1);
		  tour.setAmount(6000);

		  int tourNo = dao.saveTour(tour);
		  System.out.println("tourNo : " + tourNo);
		  
		  domesticTour = new DomesticTour();
		  domesticTour.setSource("Hyderabad");
		  domesticTour.setDestination("Delhi");
		  domesticTour.setDays(3);
		  domesticTour.setTourPackageName("Delhi Tour");
		  domesticTour.setAmount(9000);
		  domesticTour.setAdditionalStateCharges(500);
		  domesticTour.setTollTaxesToBePaid(1000);
		  int dTourNo = dao.saveTour(domesticTour);
		  System.out.println("domestic tour no : " + dTourNo);
		  
		  internationalTour = new InternationalTour();
		  internationalTour.setSource("India");
		  internationalTour.setDestination("Singapur");
		  internationalTour.setDays(10);
		  internationalTour.setTourPackageName("Singapur Tour");
		  internationalTour.setAmount(100000);
		  internationalTour.setPassportNo("P9383839");
		  internationalTour.setVisaRequired(true);
		  internationalTour.setVisaCharges(15000);
		  int iTourNo = dao.saveTour(internationalTour);
		  System.out.println("international tour no :" + iTourNo);


        Tour tour = dao.find(InternationalTour.class, 3);
        System.out.println(tour);
    }
}
```

---

# Full Example – Annotation JOINED (Table Per Subclass)

---

## Tour.java

```java
package com.annonjoined.entities;

import java.io.Serializable;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Inheritance;
import jakarta.persistence.InheritanceType;
import jakarta.persistence.Table;

import lombok.Data;

@Data
@Entity
@Table(name = "tour")
@Inheritance(strategy = InheritanceType.JOINED)
public class Tour implements Serializable {

    @Id
    @Column(name = "tour_no")
    @GeneratedValue(strategy = GenerationType.TABLE)
    protected int tourNo;

    @Column(name = "tour_package_nm")
    protected String tourPackageName;

    protected int days;
    protected String source;
    protected String destination;
    protected double amount;
}
```

---

## DomesticTour.java

```java
package com.annonjoined.entities;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.PrimaryKeyJoinColumn;

import lombok.Data;
import lombok.ToString;

@Data
@ToString(callSuper = true)
@Entity
@PrimaryKeyJoinColumn
public class DomesticTour extends Tour {

    @Column(name = "additional_state_charges")
    protected double additionalStateCharges;

    @Column(name = "toll_taxes_to_be_paid")
    protected double tollTaxesToBePaid;
}
```

---

## InternationalTour.java

```java
package com.annonjoined.entities;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.PrimaryKeyJoinColumn;

import lombok.Data;
import lombok.ToString;

@Data
@ToString(callSuper = true)
@Entity
@PrimaryKeyJoinColumn
public class InternationalTour extends Tour {

    @Column(name = "passport_no")
    protected String passportNo;

    @Column(name = "visa_required")
    protected boolean visaRequired;

    @Column(name = "visa_charges")
    protected double visaCharges;
}
```

---

## AnnonJoinedTableTest.java

```java
package com.annonjoined.test;

import com.annonjoined.dao.TourDao;
import com.annonjoined.entities.DomesticTour;
import com.annonjoined.entities.InternationalTour;
import com.annonjoined.entities.Tour;

public class AnnonJoinedTableTest {

    public static void main(String[] args) {

        TourDao dao = new TourDao();
        tour = new Tour();
		  tour.setSource("Hyderabad");
		  tour.setDestination("Warangal");
		  tour.setTourPackageName("local state tour for historic");
		  tour.setDays(1);
		  tour.setAmount(6000);

		  int tourNo = dao.saveTour(tour);
		  System.out.println("tourNo : " + tourNo);
		  
		  domesticTour = new DomesticTour();
		  domesticTour.setSource("Hyderabad");
		  domesticTour.setDestination("Delhi");
		  domesticTour.setDays(3);
		  domesticTour.setTourPackageName("Delhi Tour");
		  domesticTour.setAmount(9000);
		  domesticTour.setAdditionalStateCharges(500);
		  domesticTour.setTollTaxesToBePaid(1000);
		  int dTourNo = dao.saveTour(domesticTour);
		  System.out.println("domestic tour no : " + dTourNo);
		  
		  internationalTour = new InternationalTour();
		  internationalTour.setSource("India");
		  internationalTour.setDestination("Singapur");
		  internationalTour.setDays(10);
		  internationalTour.setTourPackageName("Singapur Tour");
		  internationalTour.setAmount(100000);
		  internationalTour.setPassportNo("P9383839");
		  internationalTour.setVisaRequired(true);
		  internationalTour.setVisaCharges(15000);
		  int iTourNo = dao.saveTour(internationalTour);
		  System.out.println("international tour no :" + iTourNo);

        Tour tour = dao.find(Tour.class, 5);
        System.out.println(tour);
    }
}
```

---

# Full Example – Annotation TABLE_PER_CLASS (Table Per Concrete Class)

---

## Tour.java

```java
package com.annontableperclass.entities;

import java.io.Serializable;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Inheritance;
import jakarta.persistence.InheritanceType;
import jakarta.persistence.Table;

import lombok.Data;

@Data
@Entity
@Table(name = "tour")
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public class Tour implements Serializable {

    @Id
    @Column(name = "tour_no")
    @GeneratedValue(strategy = GenerationType.TABLE)
    protected int tourNo;

    @Column(name = "tour_package_nm")
    protected String tourPackageName;

    protected int days;
    protected String source;
    protected String destination;
    protected double amount;
}
```

---

## DomesticTour.java

```java
package com.annontableperclass.entities;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.Table;

import lombok.Data;
import lombok.ToString;

@Data
@ToString(callSuper = true)
@Entity
@Table(name = "domestic_tour")
public class DomesticTour extends Tour {

    @Column(name = "additional_state_charges")
    protected double additionalStateCharges;

    @Column(name = "toll_taxes_to_be_paid")
    protected double tollTaxesToBePaid;
}
```

---

## InternationalTour.java

```java
package com.annontableperclass.entities;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.Table;

import lombok.Data;
import lombok.ToString;

@Data
@ToString(callSuper = true)
@Entity
@Table(name = "international_tour")
public class InternationalTour extends Tour {

    @Column(name = "passport_no")
    protected String passportNo;

    @Column(name = "visa_required")
    protected boolean visaRequired;

    @Column(name = "visa_charges")
    protected double visaCharges;
}
```

---

## AnnonTablePerClassTest.java

```java
package com.annontableperclass.test;

import com.annontableperclass.dao.TourDao;
import com.annontableperclass.entities.DomesticTour;
import com.annontableperclass.entities.InternationalTour;
import com.annontableperclass.entities.Tour;

public class AnnonTablePerClassTest {

    public static void main(String[] args) {

        TourDao dao = new TourDao();

        tour = new Tour();
		  tour.setSource("Hyderabad");
		  tour.setDestination("Warangal");
		  tour.setTourPackageName("local state tour for historic");
		  tour.setDays(1);
		  tour.setAmount(6000);

		  int tourNo = dao.saveTour(tour);
		  System.out.println("tourNo : " + tourNo);
		  
		  domesticTour = new DomesticTour();
		  domesticTour.setSource("Hyderabad");
		  domesticTour.setDestination("Delhi");
		  domesticTour.setDays(3);
		  domesticTour.setTourPackageName("Delhi Tour");
		  domesticTour.setAmount(9000);
		  domesticTour.setAdditionalStateCharges(500);
		  domesticTour.setTollTaxesToBePaid(1000);
		  int dTourNo = dao.saveTour(domesticTour);
		  System.out.println("domestic tour no : " + dTourNo);
		  
		  internationalTour = new InternationalTour();
		  internationalTour.setSource("India");
		  internationalTour.setDestination("Singapur");
		  internationalTour.setDays(10);
		  internationalTour.setTourPackageName("Singapur Tour");
		  internationalTour.setAmount(100000);
		  internationalTour.setPassportNo("P9383839");
		  internationalTour.setVisaRequired(true);
		  internationalTour.setVisaCharges(15000);
		  int iTourNo = dao.saveTour(internationalTour);
		  System.out.println("international tour no :" + iTourNo);


		/*tour = dao.find(Tour.class, 9);
		System.out.println(tour);*/
    }
}
```

# Comparison of Inheritance Strategies

## Table Per Class Hierarchy / SINGLE_TABLE

### Disadvantages

1. Data model is not normalized
2. Huge memory wastage when many subclasses exist

### Advantage

1. High performance (single table queries)

---

## Table Per Subclass / JOINED

### Disadvantages

1. More subclasses → more joined tables
2. Fetching subclass requires joins (performance impact)
3. Polymorphic queries require LEFT OUTER JOIN on all tables

### Advantage

1. Fully normalized, no memory wastage

---

## Table Per Concrete Class / TABLE_PER_CLASS

### Disadvantages

1. Change in superclass affects all subclass tables
2. High maintenance
3. Not fully normalized

### Advantage

1. No memory wastage
