
# CQRS Pattern Example: Online Food Delivery System

## Project Description
An Online Food Delivery System enables users to browse restaurants, view menus, place orders, and track delivery status. The platform must handle high-volume read operations (menu browsing, order status updates) efficiently while keeping the write operations (order placement, payment processing) consistent and isolated.

This project justifies the use of the Command Query Responsibility Segregation (CQRS) pattern due to the distinct scalability and performance needs of read and write operations.

---

## Use Case
**Place and Track Order:**
- A customer selects a restaurant, adds food items to the cart, and places an order.
- The order is processed, payment is confirmed, and the delivery status is updated.
- The customer frequently polls the system to check the order status.

This use case has a high write complexity (involving transactional operations) and high read volume (many customers checking order status), making it ideal for CQRS.

---

## Architecture Diagram
```
                        +----------------------+
                        |   API Gateway        |
                        +----------+-----------+
                                   |
                  +----------------+------------------+
                  |                                   |
        +---------v--------+                 +--------v---------+
        | Command Service  |                 |  Query Service   |
        +---------+--------+                 +--------+---------+
                  |                                   |
        +---------v--------+                 +--------v---------+
        | Write Database   |                 |   Read Database  |
        +------------------+                 +------------------+
```

---

## Example
### Command Service Endpoint
```http
POST /api/orders
```
**Payload:**
```json
{
  "customerId": "C12345",
  "restaurantId": "R98765",
  "items": [
    { "itemId": "I111", "quantity": 2 },
    { "itemId": "I222", "quantity": 1 }
  ],
  "paymentMethod": "CreditCard"
}
```

### Query Service Endpoint
```http
GET /api/orders/{orderId}/status
```
**Response:**
```json
{
  "orderId": "O45678",
  "status": "Out for Delivery"
}
```

---

## Database Design
### Write Database (Normalized)
**Orders Table:**
| OrderID | CustomerID | RestaurantID | OrderDate | PaymentStatus |
|---------|------------|--------------|-----------|---------------|

**OrderItems Table:**
| OrderItemID | OrderID | ItemID | Quantity |
|-------------|---------|--------|----------|

### Read Database (Denormalized)
**OrderStatusView:**
| OrderID | CustomerID | Status | LastUpdated |
|---------|------------|--------|-------------|

**MenuView:**
| RestaurantID | MenuItemID | Name     | Price | Description |
|--------------|------------|----------|-------|-------------|

---

## NFR (Non-Functional Requirements) Considerations
- **Scalability:** Query and command paths scale independently.
- **Performance:** Read operations use denormalized views for faster access.
- **Consistency:** Eventual consistency between write and read models via event propagation.
- **Maintainability:** Separates business logic for commands and queries.
- **Auditability:** Write model can log all state-changing actions for compliance.
- **Availability:** Query services remain available even if command services are under load.

---

## Event Propagation Mechanism
After a successful command execution (e.g., placing an order), a domain event (e.g., `OrderPlaced`) is published to a message broker (e.g., Kafka, RabbitMQ). The query service consumes this event and updates the read database accordingly.

---

## Summary
This example demonstrates how CQRS helps to optimize a high-scale online food delivery platform by:
- Ensuring fast and scalable reads.
- Isolating complex write logic.
- Supporting eventual consistency with asynchronous updates.
- Enhancing system maintainability and performance.
