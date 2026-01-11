# Customizations Available on Embedded Servlet Containers in Spring Boot

Spring Boot provides multiple ways to customize **Embedded Servlet Containers** (like Tomcat) and **DispatcherServlet** behavior.

---

## 1. Configuration-Based Customization

Using **application.yml**, we can customize:
- Embedded Servlet Container
- DispatcherServlet
- View Resolver
- Context path
- Server port

---

### Example

#### HomeController.java
```java
package com.boot.bmc.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/home")
    public String searchHomePage() {
        return "home";
    }
}
````

---

#### SpringBootMVCCustomApplication.java

```java
package com.boot.bmc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootMVCCustomApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootMVCCustomApplication.class, args);
    }
}
```

---

#### home.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <title>Home</title>
</head>
<body>
    <h2>Home</h2>
    <p>Introduced new technologly for home products.</p>
</body>
</html>
```

---

#### application.yml

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp
    servlet:
      path: /webmvc/
      load-on-startup: 1

server:
  port: 8081
  servlet:
    context-path: /mvc-custom
```

---

#### Output

```
http://localhost:8081/mvc-custom/webmvc/home
```

**Rendered Page**

```
Home
Introduced new technologly for home products.
```

---

## 2. Programmatic Customization of Embedded Server

Spring Boot allows programmatic customization in **two ways**.

---

## 2.1 Using `WebServerFactoryCustomizer`

Create a class implementing `WebServerFactoryCustomizer` and register it as a bean.

---

### Example

#### HomeController.java

```java
package com.boot.bmc.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/home")
    public String searchHomePage() {
        return "home";
    }
}
```

---

#### SpringBootMVCCustomApplication.java

```java
package com.boot.bmc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.server.ConfigurableWebServerFactory;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class SpringBootMVCCustomApplication {

    @Bean
    public WebServerFactoryCustomizer<ConfigurableWebServerFactory> webServerFactoryCustomizer() {
        return new BootMvcWebCustomizer();
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringBootMVCCustomApplication.class, args);
    }

    private final class BootMvcWebCustomizer
            implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {

        @Override
        public void customize(ConfigurableWebServerFactory factory) {
            factory.setPort(8082);
        }
    }
}
```

---

### Converted to Lambda Expression

```java
@Bean
public WebServerFactoryCustomizer<ConfigurableWebServerFactory> webServerFactoryCustomizer() {
    return (ConfigurableWebServerFactory factory) -> {
        factory.setPort(8082);
    };
}
```

---

#### home.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <title>Home</title>
</head>
<body>
    <h2>Home</h2>
    <p>Introduced new technologly for home products.</p>
</body>
</html>
```

---

#### application.yml

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp
```

---

#### Output

```
http://localhost:8082/home
```

**Rendered Page**

```
Home
Introduced new technologly for home products.
```

---

## 2.2 Using `ConfigurableServletWebServerFactory` (Most Used)

This approach gives **full control** over embedded server configuration.

---

### Example

#### HomeController.java

```java
package com.boot.bmc.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/home")
    public String searchHomePage() {
        return "home";
    }
}
```

---

#### SpringBootMVCCustomApplication.java

```java
package com.boot.bmc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class SpringBootMVCCustomApplication {

    @Bean
    public ConfigurableServletWebServerFactory configurableServletWebServerFactory() {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
        factory.setPort(8083);
        factory.setContextPath("/mvc-custom");
        return factory;
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringBootMVCCustomApplication.class, args);
    }
}
```

---

#### home.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <title>Home</title>
</head>
<body>
    <h2>Home</h2>
    <p>Introduced new technologly for home products.</p>
</body>
</html>
```

---

#### application.yml

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp
```

---

#### Output

```
http://localhost:8083/mvc-custom/home
```

**Rendered Page**

```
Home
Introduced new technologly for home products.
```

---

## Summary

| Approach                            | Customization Type | Usage                    |
| ----------------------------------- | ------------------ | ------------------------ |
| application.yml                     | Declarative        | Simple & preferred       |
| WebServerFactoryCustomizer          | Programmatic       | Medium-level control     |
| ConfigurableServletWebServerFactory | Programmatic       | Full control (Most Used) |

---
