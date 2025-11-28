# Spring Boot Executable Jars

## How do we distribute Spring Boot applications?

- In general, we can distribute:
  - A Java application as a **JAR**
  - A web application as a **WAR**

But in the case of **Spring Boot**, both Java applications and web applications are distributed as **JAR** only.

```

Java Application  = JAR
Web Application   = Deploy on server = WAR

````

➡️ **Both Java and Web applications are distributed as JAR in Spring Boot.**

---

## How do we distribute a Java application to the end-user?

By packaging it as a **JAR**, but there are **2 types of JARs**:

1. **Distributable JAR** – Java libraries are packaged  
2. **Executable JAR** – Java application is packaged

### Difference

- If **main method is NOT present** in `MANIFEST.MF` → **Distributable JAR**  
- If **main method IS present** in `MANIFEST.MF` → **Executable JAR**

---

## Example - 1

```java
class A {
  public static void main(String[] args) {}
}
````

### Compile

```
javac A.java
```

### Create the JAR file

```
jar -cvf app.jar A
```

### Run (complex for end-user)

```
java -jar -cp d:/app.jar A
```

The end user must know the class containing the main method — not convenient.

---

## Making an Executable JAR

To fix the above issue, we create an **executable JAR** by adding the main class into manifest.

### Step 1: Create `manifest.txt`

```
Main-Class: A
```

### Step 2: Create the JAR using manifest

```
jar -cvfm manifest.txt app.jar *.class
```

---

## JAR Structure

```
app.jar
 ├─ META-INF
 │    └─ MANIFEST.MF
 └─ A.class
```

### Run

```
java -jar app.jar
```

Now it works because JVM reads the `Main-Class` entry from `MANIFEST.MF`.


# Example - 2

## Spring JARs
- spring-core  
- spring-context  
- spring-context-support  
- spring-beans  

```java
@Component
public class Toy {
  public void play() {
       System.out.println("Playing....");
  }
}

public class Application {
  public static void main(String[] args) {
    ApplicationContext context = new AnnotationConfigApplicationContext("package name");
    Toy toy = context.getBean("toy", Toy.class);
    toy.play();
  }
}
````

### Compile the Java program

Build classpath pointing to all the Spring Framework JARs:

```
javac Toy.java
javac Application.java
```

**bin/**

```
Toy.class
Application.class
```

### manifest.txt

```
Main-Class: Application
```

### Create the JAR

```
jar -cvfm spring-app.jar manifest.txt *.class
```

### Run the JAR

```
java -jar spring-app.jar
```

> JVM will read `manifest.mf` and find the main class, but the application **will not work** because Spring JARs are missing in the classpath.

---

## JAR Structure

```
spring-app.jar
   ├─ META-INF
   │    └─ manifest.mf
   ├─ Toy.class
   └─ Application.class
```

---

## Why classpath does NOT work with `java -jar`

Trying:

```
java -cp lib/spring-core-5.0.jar;lib/spring-context-5.0.jar -jar spring-app.jar
```

or

```
set classpath=lib/spring-core-5.0.jar;lib/spring-context-5.0.jar
java -jar spring-app.jar
```

**Both will NOT work.**

### Reason

* An **executable JAR** is an **isolated JAR**.
* JVM loads everything only from inside `manifest.mf`.
* JVM *ignores external classpath* when `-jar` option is used.
* So if your executable JAR depends on external JARs, those dependencies **must be listed inside the manifest file**.

---

## Project Structure

```
d:\spring-app
       ├─ src
       │    └─ *.java
       ├─ lib
       │    ├─ spring-core-5.0.RELEASE.jar
       │    └─ spring-context-5.0.RELEASE.jar
       └─ bin
            └─ spring-app.jar
```

### manifest.txt

```
Main-Class: Application
Class-Path: d:/lib/spring-core-5.0.RELEASE.jar;d:/lib/spring-context-5.0.RELEASE.jar
```

### Inside spring-app.jar

```
spring-app.jar
   ├─ META-INF
   │    └─ manifest.mf
   │         Main-Class: Application
   │         Class-Path: d:/lib/spring-core-5.0.RELEASE.jar;d:/lib/spring-context-5.0.RELEASE.jar
   ├─ Toy.class
   └─ Application.class
```

⚠️ **But this still will NOT work — absolute paths are not allowed in manifest.mf.**

---

## Correct Approach

Place all Spring JARs and your executable JAR in the **same folder**:

```
dist
   ├─ spring-core-5.0.RELEASE.jar
   ├─ spring-context-5.0.RELEASE.jar
   └─ spring-app.jar
```

### manifest.txt (Correct)

```
Main-Class: Application
Class-Path: ./spring-core-5.0.RELEASE.jar spring-context-5.0.RELEASE.jar
```

### Create JAR

```
jar -cvfm manifest.txt spring-app.jar *.class
```

### Run the JAR

```
java -jar spring-app.jar
```

✅ Now it will work.

---

## spring-app.jar Structure

```
spring-app.jar
   ├─ META-INF
   │    └─ manifest.mf
   ├─ Toy.class
   └─ Application.class
```


# Final Example

## Place Spring JARs

Before creating classes, make sure to place the Spring jars in the following location:

```
D:\
└─Core Java
    └─dist
        └─lib
            ├─spring-context-6.1.6.jar
            ├─spring-core-6.1.6.jar
            ├─spring-beans-6.1.6.jar
            ├─commons-logging-1.2.jar
            ├─spring-aop-6.1.6.jar
            ├─spring-expression-6.1.6.jar
```

---

## Create a **dist** Folder and Add Below Classes

### Toy.java

```java
package com.blc.beans;

public class Toy {

    public void play()
    {
        System.out.println("Playing...");
    }
}
```

### JavaConfig.java

```java
package com.blc.beans;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = {" com.blc.beans "})
public class JavaConfig {
    @Bean
    public Toy toy()
    {
        Toy toy = new Toy();
        return toy;
    }
}
```

### Application.java

```java
package com.blc.beans;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Application {

    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(JavaConfig.class);
        Toy toy = context.getBean("toy", Toy.class);
        toy.play();
    }
}
```

---

## Compile the Spring Application

Run the following command:

```
D:\Core java\dist>javac -d . -cp "lib/*" *.java
```

---

## Create `manifest.txt`

### manifest.txt

```
Main-Class: com.blc.beans.Application
Class-Path: lib/spring-context-6.1.6.jar lib/spring-core-6.1.6.jar lib/spring-beans-6.1.6.jar lib/commons-logging-1.2.jar lib/spring-aop-6.1.6.jar lib/spring-expression-6.1.6.jar
```

*(After the last jar, press **Enter**)*

---

## Create the JAR File (`spring-app.jar`)

```
D:\Core java\dist>jar -cvfm spring-app.jar manifest.txt com\blc\beans\*.class
```

---

## Verify the Created JAR File

```
D:\Core java\dist>jar -tvf spring-app.jar
```

### Output:

```
   0 Fri Nov 28 11:26:36 IST 2025 META-INF/
 292 Fri Nov 28 11:26:36 IST 2025 META-INF/MANIFEST.MF
 671 Fri Nov 28 11:17:22 IST 2025 com/blc/beans/Application.class
 577 Fri Nov 28 11:17:22 IST 2025 com/blc/beans/JavaConfig.class
 399 Fri Nov 28 11:17:22 IST 2025 com/blc/beans/Toy.class
```

---

## Run the Executable JAR

```
D:\Core java\dist>java -jar spring-app.jar
```

### Output:

```
Playing...
```

---

## spring-app.jar Structure

```
spring-app.jar
   ├─META-INF
   │   └─manifest.mf
   ├─Toy.class
   ├─Application.class
   └─JavaConfig.class
```

---

## Note

Without creating the jar file multiple times, you can test using:

```
java -cp "spring-app.jar;lib/*" com.blc.beans.Application
```


