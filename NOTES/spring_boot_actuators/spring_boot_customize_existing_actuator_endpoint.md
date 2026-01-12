# Customizing Existing Actuator Endpoints (Spring Boot Actuator 2 & 3)

---

## Example: Customize the Existing `info` Endpoint

### CustomInfoEndPoint.java
```java
package com.ba.management.endpoint;

import org.springframework.boot.actuate.endpoint.annotation.ReadOperation;
import org.springframework.boot.actuate.endpoint.web.annotation.EndpointWebExtension;
import org.springframework.boot.actuate.info.InfoEndpoint;
import org.springframework.stereotype.Component;

import java.util.Properties;

@Component
@EndpointWebExtension(endpoint = InfoEndpoint.class)
public class CustomInfoEndPoint {

    @ReadOperation
    public Properties info() {
        Properties props = new Properties();
        props.put("application-name", "Boot MVC Application");
        props.put("version", "1.0");
        props.put("autor", "gireesh");
        props.put("license", "open gpl license");
        return props;
    }
}
````

---

### application.yml

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp

management:
  endpoints:
    web:
      exposure:
        include:
          - health
          - info
          - shutdown
          - features

  endpoint:
    shutdown:
      enabled: true
```

---

### SpringBootActuatorsApplication.java

```java
package com.ba;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootActuatorsApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootActuatorsApplication.class, args);
    }
}
```

---

### Output (Custom Info Endpoint)

* Access URL (GET):

```
http://localhost:8080/actuator/info
```

* Response:

```json
{
  "license": "open gpl license",
  "version": "1.0",
  "applicationName": "Boot MVC Application",
  "autor": "gireesh"
}
```

---

## Custom Health Endpoint Example

### CustomHealthEndPoint.java

```java
package com.ba.management.endpoint;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class CustomHealthEndPoint implements HealthIndicator {

    @Override
    public Health health() {
        return Health.up()
                .withDetail("application", "Running")
                .withDetail("version", "1.0")
                .withDetail("authour", "gireesh")
                .build();
    }
}
```

---

### application.yml (Health Details Enabled)

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp

management:
  endpoints:
    web:
      exposure:
        include:
          - health
          - info
          - shutdown
          - features

  endpoint:
    health:
      show-details: always
```

---

### Output (Custom Health Endpoint)

* Access URL:

```
http://localhost:8080/actuator/health
```

* Response:

```json
{
  "status": "UP",
  "components": {
    "customHealthEndPoint": {
      "status": "UP",
      "details": {
        "application": "Running",
        "version": "1.0",
        "authour": "gireesh"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 629144547328,
        "free": 188405108736,
        "threshold": 10485760,
        "path": "D:\\Core java\\sspringboot\\spring-bbot-actuator\\spring-boot-actuators\\spring-boot-actuators\\.",
        "exists": true
      }
    },
    "ping": {
      "status": "UP"
    },
    "ssl": {
      "status": "UP",
      "details": {
        "validChains": [],
        "invalidChains": []
      }
    }
  }
}
```
