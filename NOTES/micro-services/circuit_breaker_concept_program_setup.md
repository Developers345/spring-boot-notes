# Circuit Breaker
----------------

→ **Circuit Breaker** is an **integration-tier design pattern** and a **client-side design pattern**.

→ It is used to **protect a system from failures and performance degradation**
when a dependent service becomes slow or unavailable.

---

## Key Concepts
--------------

### Failure Threshold
- **failure.threshold** = maximum number of failures permitted for a service
- Example:
  ```text
  failure.threshold = 5



## Client-Side Implementation

---

→ Circuit Breaker is implemented **at the client side**, not on the server.

→ When a service receives **too many requests**, it may:

* Slow down
* Stop responding
* Go down completely

→ To protect the client and the overall system, Circuit Breaker is applied
**before invoking the remote service**.

---

## Working of Circuit Breaker

---

1. Client sends requests to the service.
2. If the service does not respond or fails:

   * Failure count is incremented.
3. When failures reach the configured limit:

   ```text
   failure.threshold = 5
   ```

   * Circuit Breaker **opens the circuit**.
4. Once the circuit is open:

   * All further requests **fail immediately at the client side**
   * No calls are made to the server
5. Circuit remains open for a configured duration:

   ```text
   open.timeout = 5 ms
   ```
6. During this timeout:

   * Server gets time to recover
   * Ongoing requests are processed slowly
7. After timeout expires:

   * Client retries sending requests
   * If service responds successfully, circuit closes
   * If failures continue, circuit re-opens

---

## Circuit Breaker States

---

* **CLOSED**

  * Requests flow normally
  * Failures are monitored

* **OPEN**

  * Requests are blocked
  * Immediate failure response to client

* **HALF-OPEN**

  * Limited test requests are allowed
  * Success → move to CLOSED
  * Failure → move to OPEN

---

## Circuit Breaker Scope

---

→ **One Circuit Breaker instance is created per service invocation**
(or per downstream service).

---

## Customization in Spring Boot

---

→ Circuit Breaker objects are created using **auto-configuration**.

→ To customize behavior:

* Create a **Customizer class**
* Register it as a bean in the **IoC container**

→ After the IoC container creates the Circuit Breaker instance,
it passes that instance to the **Customizer**, allowing us to configure:

* Failure threshold
* Timeout duration
* Sliding window size
* Time limiter

---

## Key Takeaways

---

* Circuit Breaker is a **client-side resilience pattern**
* Prevents cascading failures
* Fails fast instead of waiting for timeouts
* Improves system stability and responsiveness
* Works best with:

  * Timeouts
  * Retries
  * Fallback mechanisms

---
## Pictorial Respresentation
<img width="1919" height="720" alt="Screenshot 2026-01-16 173939" src="https://github.com/user-attachments/assets/5ceed10d-eba8-4b89-bc12-f968a22608a2" />
