# 1. javac command 
We can use `javac` command to compile a single or group of Java source files.  
To compile single or multiple Java source files:

## Syntax:
```

javac [options] <java_source_files>

javac [options] Test.java
javac [options] A.java B.java C.java
javac [options] *.java

```

### Common Options:
- `-version`
- `-d`
- `-source`
- `-verbose`
- `-cp` / `-classpath` : setting classpath

---

# java command
We can use `java` command to run a single class file.

## Syntax:
```

java [options] Test A B C

```
Here `A B C` are command-line arguments.

### Common Options:
- `-version`
- `-D` : set system property
- `-cp` / `-classpath` : set classpath  
- `-ea` / `-esa` / `-dsa` / `-da` : enable/disable assertions

### Note:
- We can compile **any number of source files** at a time.
- But you can run **only one class file** at a time.

---

# What is classpath?
- Classpath describes the location where required `.class` files are available.
- Java compiler and JVM use the classpath to locate required `.class` files.
- By default, JVM always searches the **current directory** for `.class` files.
- If JVM does not find the `.class` file in the current working directory, it throws  
  **`java.lang.ClassNotFoundException`**.
- If we set the classpath explicitly, JVM will search only in the specified classpath and **not** in the current directory.

---

# Ways to set the classpath

## 1. Using environment variable `CLASSPATH`
This way is **permanent** and preserved across system restarts.  
Recommended when installing permanent software.

---

## 2. At command prompt level  
```

set classpath=C:\core_java

```
- This is temporary.
- Works only for the current command prompt session.

---

## 3. At command level using `-cp`
```

java -cp C:\core_java Test

```
- This applies only for that particular command.
- After command execution, the classpath is lost.

---

### Once classpath is set you can run the program from any location.

Example:
```

java -cp "D:\Core java\jar\java_classpath_pratice" Test

```

### Important:
Once we set the classpath, **JVM will NOT search the current working directory**.  
It searches only in the provided classpath.

Example (throws error):
```

D:\Core java\jar\java_classpath_pratice> java -cp C: Test

```
Output: `java.lang.ClassNotFoundException`

### How to tell JVM to check current directory first?
Use `.` in classpath:

```

java -cp .;C: Test

````
This works because:
- `.` = current directory
- `C:` = alternate path

---

# Example Program
```java
public class Test {
    public static void main(String[] args) {
        System.out.println("Classpath Demo");
    }
}
````

---

# Note

Among the three ways of setting classpath,
**setting classpath at command level (`-cp`) is recommended**
because dependent classes may vary from command to command.


# Example : C: (C drive)
```java
public class AStudent
{
  public void m1()
  {
    System.out.println("Job Immediately");
  }
}
````

---

# D: (D drive)

```java
public class ITIndustry
{
  public static void main(String[] args)
  {
    AStudent student = new AStudent();
    student.m1();
    System.out.println("You will get Soon");
  }
}
```

---

# Compilation & Execution

```
C:/> javac AStudent.java
D:/> javac ITIndustry.java
```

❌ **Exception will come** (because AStudent.class not found)

---

### Correct Compilation

```
D:/> javac -cp C: ITIndustry.java     (It will work)
```

---

# Execution Attempts

```
D:/> java ITIndustry
```

❌ **Exception: AStudent class not found**

```
D:/> java -cp C: ITIndustry
```

❌ **Exception: ITIndustry class not found**

---

### Correct Execution

```
D:/> java -cp .;C: ITIndustry      (runs without error)
```

Or from another drive:

```
E:/> java -cp D:;C: ITIndustry     (runs without error)
```

---

# Important Points

* **Compiler** checks **one level of dependency only**.
* **JVM** checks **all levels of dependencies**.
* In **classpath**, the **order of locations is important**.
* JVM always checks classpath **from left to right** until the required class is found.


# JAR File
- All third-party software libraries are usually available in the form of **JAR files**.

## Example
- To develop servlets, all required dependent classes are available in `servlet-api.jar`.  
  We must place this JAR file in the classpath to compile a servlet program.
- To run a JDBC program, dependent classes are available in `ojdbc14.jar`.  
  We must place this JAR file in the classpath.
- To use Log4j, all required classes are present in `log4j.jar`.  
  We must place this JAR in the classpath; otherwise, Log4j-based applications won't run.

---

# Various Commands in JAR File

## 1. Create a JAR file (Zip file)
```

jar -cvf durgacalcu.jar Test.class
jar -cvf durgacalcu.jar A.class B.class C.class
jar -cvf durgacalcu.jar *.class
jar -cvf durgacalcu.jar *.*

```

**Options:**
- `c` → Create  
- `v` → Verbose  
- `f` → Specify JAR file name  

---

## 2. Extract a JAR file (Unzip)
```

jar -xvf durgacalcu.jar

```
Options:
- `x` → Extract  
- `v` → Verbose  
- `f` → Named file  

---

## 3. Display table of contents
```

jar -tvf durgacalcu.jar

````
Options:
- `t` → Table of contents  
- `v` → Verbose  
- `f` → Named file  

---

# Example

Chip.java coming from service provider as a JAR.

## Chip.java
```java
public class Chip {

    public void read() {
        System.out.println("Chip is Reading..");
    }
}
````

→ Compile Chip.java, create a JAR, and place it anywhere on your system.

## Robot.java

```java
public class Robot {
    public static void main(String[] args) {
        Chip chip = new Chip();
        chip.read();
        System.out.println("Robot Starting..");
    }
}
```

### How to run Robot class:

```
D:\Core java\jar\java_classpath_pratice> java -cp .;"D:\Core java\chip.jar" Robot
```

### Output:

```
Chip is Reading..
Robot Starting..
```

---

# Shortcut: Place JAR in Extension Directory

If a JAR file is placed here:

```
JDK
 └─ jre
      └─ lib
           └─ ext
               └─ *.jar
```

Then all classes inside the JAR become available to the compiler and JVM without setting the classpath.

---

# System Properties

* Every system maintains persistent information as **System Properties**:

  * JVM vendor
  * Java version
  * User country
  * OS details
  * Classpath info
  * etc.

## Example Program

```java
import java.util.*;

public class GetAllSystemProperties {
    public static void main(String[] args) {
        Properties p = System.getProperties();
        p.list(System.out);
    }
}
```

---

# Output (Sample)

```
-- listing properties --
java.specification.version=17
sun.cpu.isalist=amd64
sun.jnu.encoding=Cp1252
java.class.path=.
java.vm.vendor=Oracle Corporation
...
java.class.version=61.0
```

---

# How to set System Property from Command Prompt

```
java -Ddurga=ocjp Test
```

* `-D` → sets a system property.

### Benefit:

* Used to customize behavior of a Java program.

---

# Customized System Property Example

```java
public class CustomizeSystemProperty {
    public static void main(String[] args) {
        String course = System.getProperty("course");

        if (course.equals("scjp")) {
            System.out.println("scjp information");
        } else {
            System.out.println("Other Information");
        }
    }
}
```

### Output:

```
D:\> java -Dcourse=scjp CustomizeSystemProperty
scjp information

D:\> java -Dcourse=sert CustomizeSystemProperty
Other Information
```

---

# jar vs war vs ear

## JAR (Java Archive)

* Contains a group of `.class` files.

## WAR (Web Archive)

* Represents a **web application**.
* Contains: servlets, JSPs, HTML, JS, CSS, images, etc.
* Advantage: easy deployment, delivery, transportation.

## EAR (Enterprise Archive)

* Represents an **enterprise application**.
* Contains: servlets, JSPs, EJBs, JMS components, etc.

**Note:**
EAR = multiple WAR files + JAR files.

---

# Web Application vs Enterprise Application

## Web Application

* Developed using: servlets, JSP, HTML, CSS, JS, etc.
* Examples:

  * Online Library System
  * Online Shopping Cart

## Enterprise Application

* Developed using full J2EE stack: servlets, JSP, EJB, JMS, etc.
* Examples:

  * Banking Application
  * Telecom Billing System

**Note:**
J2EE / JEE compatible applications are enterprise applications.

---

# Web Server vs Application Server

## Web Server

* Runs **web applications**.
* Supports web technologies (Servlets, JSP, HTML).
* Example: **Tomcat**

## Application Server

* Runs **enterprise applications**.
* Supports entire J2EE stack (Servlets, JSP, EJB, JMS, etc.)
* Examples:

  * WebLogic
  * WebSphere
  * JBoss

**Note:**

* Every application server contains a built-in web server.
* J2EE compatible server = Application Server.

---

# How to Create Executable JAR Files

## ExectuableJarTest.java

```java
public class ExectuableJarTest {
    public static void main(String[] args) {
        System.out.println("I am executable jar....");
    }
}
```

## manifest.MF

```
Main-Class: ExectuableJarTest

```

(**Must press ENTER at the end**)

## Create JAR

```
jar -cvfm executablejar.jar manifest.MF ExectuableJarTest.class
```

## Run JAR

```
java -jar executablejar.jar
```

Output:

```
I am executable jar....
```

---

# How to Create Dependent Executable JAR

## Chip.java

```java
public class Chip {
    public void read() {
        System.out.println("Chip is Reading..");
    }
}
```

→ Compile and place the JAR inside `lib` directory.

## Robot.java

```java
public class Robot {
    public static void main(String[] args) {
        Chip chip = new Chip();
        chip.read();
        System.out.println("Robot Starting..");
    }
}
```

## manifest.MF

```
Main-Class: Robot
Class-Path: lib/chip.jar

```

(ENTER required at end)

## Create JAR

```
jar -cvfm robot.jar manifest.MF Robot.class
```

## Run JAR

```
java -jar robot.jar
```

### Output:

```
Chip is Reading..
Robot Starting..
```

# **Below is the **WAR (Web Application Archive)** file directory structure explained in a clear and detailed way.**

---

# **WAR File Directory Structure in Java (Standard Java Web Application)**

A **WAR file** is a packaged web application used for deployment on Java application servers like **Tomcat**, **Jetty**, **JBoss**, **WebLogic**, etc.
Internally, a WAR file follows a **well-defined directory structure**.

---

# **Top-Level Structure of a WAR File**

```
yourapp.war
│
├── index.jsp
├── login.jsp
├── styles.css
├── images/
│   └── logo.png
├── scripts/
│   └── main.js
│
└── WEB-INF/
    ├── web.xml
    ├── classes/
    │   ├── com/
    │   │   └── example/
    │   │       └── MyServlet.class
    │   └── application.properties
    │
    ├── lib/
    │   ├── spring-core.jar
    │   ├── mysql-connector.jar
    │   └── log4j.jar
    │
    └── tlds/
        └── custom.tld
```

---

# **Explanation of WAR Structure**

## **1. Root Directory (Public Web Content)**

Everything **outside `WEB-INF`** is publicly accessible.

| Folder/File          | Description        |
| -------------------- | ------------------ |
| `index.jsp`, `*.jsp` | Public JSP pages.  |
| `images/`            | Images used in UI. |
| `css/` or `styles/`  | CSS files.         |
| `js/` or `scripts/`  | JavaScript files.  |
| `html/`              | Public HTML pages. |

---

# **2. `WEB-INF` (Secure Area – Not Publicly Accessible)**

Everything inside `WEB-INF` **cannot be accessed directly via browser**.

### **Important items inside `WEB-INF`:**

## **a) `web.xml`**

Deployment descriptor file.

```
WEB-INF/web.xml
```

Defines:

* servlet mappings
* filters
* listeners
* welcome files
* security configs

---

## **b) `classes/`**

Contains:

* compiled `.class` files
* configuration properties (`.properties`, `.xml`)
* packages such as `com/example/...`

Example:

```
WEB-INF/classes/com/example/MyServlet.class
WEB-INF/classes/application.properties
```

---

## **c) `lib/`**

All dependent JAR libraries of the application.

Example:

```
WEB-INF/lib/mysql-connector.jar
WEB-INF/lib/spring-web.jar
WEB-INF/lib/jackson.jar
```

These JARs are automatically added to the application classpath.

---

## **d) `tlds/` (optional)**

Tag Library Descriptor files for custom JSP tags.

```
WEB-INF/tlds/mytags.tld
```

---

# **Full WAR Layout Summary**

```
ROOT
│
├── Public files (JSP/HTML/CSS/JS/Images)
│
└── WEB-INF
     │
     ├── web.xml               (Servlet config)
     ├── classes/              (Compiled classes)
     ├── lib/                  (3rd party JARs)
     ├── tlds/                 (Custom JSP tags)
     └── application configs
```

---

# **How WAR is Created?**

Using Maven:

```
mvn clean package
```

Outputs:

```
target/yourapp.war
``
