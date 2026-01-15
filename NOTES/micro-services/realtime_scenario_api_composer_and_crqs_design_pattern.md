# Real-Time Scenario: Querying Data Across Multiple Microservices

## Business Scenario: E-Commerce Application

Consider a **large-scale E-Commerce system** built using **Microservices Architecture**.

### Microservices in the System
- **Order Service** – manages orders
- **Customer Service** – manages customer details
- **Product Service** – manages product information
- **Payment Service** – manages payment status
- **Shipping Service** – manages delivery details

---

## Requirement 1: Live Order Details Screen (API Composer)

### Use Case
When a **customer opens the “My Orders” page**, the system must show **real-time order information**, such as:
- Order status (Placed / Shipped / Delivered)
- Payment status (Success / Pending)
- Current shipping location
- Product details

### Solution: API Composer (Aggregator)

- A **single REST endpoint** called `OrderSummary API` is created.
- This API calls multiple microservices **synchronously**.
- Data is aggregated **at runtime** and returned to the UI.

### Flow
```

Client (My Orders Page)
↓
Order Summary API (API Composer)
↓
-

| Order Service | Payment Service | Shipping   |
| Product       | Customer        |            |
------------------------------------------------

```
    ↓
```

Aggregated Live Response

```

### Why API Composer?
- Data must be **live and accurate**
- Customer expects **real-time updates**
- Example:
  - Payment just completed
  - Order status just changed to *Shipped*

### Drawback in This Scenario
- Every request triggers **multiple service calls**
- Heavy **CPU and memory usage**
- **Performance degrades** during high traffic (festival sales)

---

## Requirement 2: Sales Dashboard & Reports (CQRS)

### Use Case
Business users want a **Sales Dashboard** showing:
- Total orders per day
- Revenue per product
- Top customers
- Monthly / yearly sales trends

⚠️ **Live data is NOT required**  
✔ **Fast performance is required**

---

## Solution: CQRS (Command Query Responsibility Segregation)

- Each microservice **publishes events**:
  - `OrderCreatedEvent`
  - `PaymentCompletedEvent`
  - `ProductPurchasedEvent`
- CQRS consumes these events.
- Data is stored in a **Read-Only / View Database**.
- Dashboard queries are served from this database.

### Flow
```

Order / Payment / Product Services
↓ (Events)
CQRS
↓
Read-Only (View) Database
↓
Dashboard / Reports

```

### Why CQRS?
- Data is **pre-aggregated**
- No runtime service-to-service calls
- Extremely **fast queries**
- Scales well for **analytics and reporting**

### Limitation
- Data is **eventually consistent**
- Shows **historical data**, not live updates

---

## Comparison in Real-Time Context

| Requirement                | API Composer               | CQRS                         |
|---------------------------|----------------------------|------------------------------|
| Customer Order Tracking   | ✅ Best Choice              | ❌ Not suitable               |
| Sales Dashboard           | ❌ Poor performance         | ✅ Best Choice                |
| Live Data Needed          | Yes                        | No                           |
| Query Performance         | Medium to Low              | Very High                    |
| Scalability               | Limited                    | High                         |

---

## Final Conclusion

- **API Composer** is ideal for:
  - Customer-facing **real-time screens**
  - Live order tracking

- **CQRS** is ideal for:
  - **Dashboards**
  - **Reports**
  - **Analytics**
  - High-read, low-latency systems

👉 In real-world systems, **both patterns are used together** to balance **consistency, performance, and scalability**.
```
