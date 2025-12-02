# Application Configuration

## How can we manage the application configuration while working with Spring Boot?  
### (or)  
## What are the configuration values being detected and loaded into IOC Container by the `SpringApplication` class?

---

### ➤ Externalizing Configuration

Spring Boot lets you **externalize the Application Configuration** so that we can work with multiple different environments.  
You can place configuration in:

- `application.properties`
- `application.yaml`
- Environment variables
- System properties
- Command-line arguments

All of these will be detected and loaded into the **Environment object** by `SpringApplication`.

---

### ➤ Why Externalize Configuration?

To ensure Spring Boot applications work across different environments without hardcoding values, Spring Boot allows us to **externalize the configuration**, which helps initialize application components dynamically.

---

## Order of Configuration Loading in Spring Boot

Spring Boot looks for configuration values in the **following order (highest priority to lowest)**:

1. **DevTools properties**  
   If DevTools are enabled, it reads:  
   `~\spring-boot-devtools.properties`

2. **Environment variable `SPRING_APPLICATION_JSON`**  
   Inline JSON key-value pairs.

3. **ServletConfig / ServletContext init parameters**

4. **Environment variables**

5. **System properties**

6. **RandomValuePropertySource**  
   Can be used in properties files using `random.*`

7. **application.properties / application.yaml** (in following search order):  
   - `Current project/config/`  
   - `Current project/`  
   - `Classpath/config/`  
   - `Classpath (resources folder)`

---

## Examples

### **1. Using `SPRING_APPLICATION_JSON` environment variable**

```cmd
set SPRING_APPLICATION_JSON={"motorNo":"123","type":"Electric","warranty":"2Months","horsePower":"1200MP","fuelType":"Disel"}
````

SpringApplication loads these into the **Environment object**.

Check the value (Windows):

```cmd
echo %SPRING_APPLICATION_JSON%
```

---

### **2. Environment Variable Example**

```
fuelType=Disel
```

SpringApplication will load this property into the Environment.

---

### **3. System Property Example**

```cmd
java -jar -DhorsePower=1500MP target\boot-external-configuration-1.0.0-SNAPSHOT.jar
```

This value will also be loaded by SpringApplication.

---

## Locations Where `application.properties` / `application.yaml` Are Detected

All the following locations work:

1. **Current project/config**
   Example:

   ```
   D:\Core java\sspringboot\boot-external-configuration\target\config\application.properties
   ```

2. **Current project**

   ```
   D:\Core java\sspringboot\boot-external-configuration\target\application.properties
   ```

3. **Classpath/config**
   (Inside `src/main/resources/config`)

4. **Classpath/resources**
   (Inside `src/main/resources`)

---
