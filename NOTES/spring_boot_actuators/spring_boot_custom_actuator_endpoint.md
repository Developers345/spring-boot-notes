# Custom Actuator Endpoint Example

---

## Feature.java
```java
package com.ba.management.endpoint;

public class Feature {

    private String featureName;
    private String description;

    public Feature() {
    }

    public Feature(String featureName, String description) {
        this.featureName = featureName;
        this.description = description;
    }

    public String getFeatureName() {
        return featureName;
    }

    public void setFeatureName(String featureName) {
        this.featureName = featureName;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
````

---

## FeatureEndpoint.java

```java
package com.ba.management.endpoint;

import org.springframework.boot.actuate.endpoint.annotation.Endpoint;
import org.springframework.boot.actuate.endpoint.annotation.ReadOperation;
import org.springframework.boot.actuate.endpoint.annotation.WriteOperation;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

/**
 * Technology-agnostic endpoint we can create using Spring Boot 2 Actuator
 * 
 * @author Gireesh
 */
@Component
@Endpoint(id = "features")
public class FeatureEndpoint {

    private List<Feature> features;

    public FeatureEndpoint() {
        features = new ArrayList<>();
        Feature feature = new Feature(
                "cancel-incident",
                "Customer raise the request for cancel feature not more used"
        );
        features.add(feature);
    }

    @ReadOperation
    public List<Feature> getFeatures() {
        return features;
    }

    @WriteOperation
    public Feature addFeature(String featureName, String description) {
        Feature feature = new Feature(featureName, description);
        features.add(feature);
        return feature;
    }
}
```

---

## SpringBootActuatorsApplication.java

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

## application.yml

```yaml
spring:
  mvc:
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp

management:
  endpoints:
#   enabled-by-default: false
    web:
      exposure:
        include:
          - health
          - shutdown
          - beans
          - configprops
          - logfiles
          - loggers
          - threaddump
          - features   # custom endpoint
        exclude:
          - info
          - env
  endpoint:
    shutdown:
      enabled: true
```

---

## Output

* We can access the custom actuator endpoint using the following URLs:

```
GET  : http://localhost:8080/actuator/features
POST : http://localhost:8080/actuator/features
```

---

## Spring Boot 2 & 3 Actuator – Key Notes

* Write a **component class (Spring Bean)** and annotate it with:

  ```java
  @Endpoint(id = "endpointName")
  ```

  * The `id` becomes the **endpoint URL** used for accessing the actuator endpoint.

* Write methods that may:

  * Take parameters

  * Return values
    These methods must be annotated with one of the following:

  1. `@ReadOperation`

     * Mapped to **HTTP GET** request (for `web.exposure` endpoints)

  2. `@WriteOperation`

     * Mapped to **HTTP POST** request

  3. `@DeleteOperation`

     * Mapped to **HTTP DELETE** request

* Data is **sent and received in JSON format**.

