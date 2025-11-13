# Event-Driven Architecture

## Overview

Event-Driven Architecture (EDA) is a software design pattern where components communicate through the production, detection, and consumption of events. An event represents a significant change in state.

## Structure

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│  Producer   │──Event─▶│Event Broker │──Event─▶│  Consumer   │
│  Service    │         │ (Event Bus) │         │  Service    │
└─────────────┘         └─────────────┘         └─────────────┘
                               │
                        ┌──────┴──────┐
                        │             │
                   Consumer A    Consumer B
```

## Key Concepts

### Events
- Immutable records of something that happened
- Past tense naming (OrderPlaced, UserRegistered)
- Contains relevant data about the state change

### Event Producers
- Components that detect changes and publish events
- Don't know who will consume the events
- Decoupled from consumers

### Event Consumers
- Subscribe to events they're interested in
- React to events independently
- Can have multiple consumers for one event

### Event Broker/Bus
- Middleware that routes events
- Examples: Kafka, RabbitMQ, AWS EventBridge
- Handles delivery, persistence, ordering

## Patterns

### Pub/Sub (Publish-Subscribe)
```javascript
// Publisher
eventBus.publish('order.placed', {
  orderId: '123',
  customerId: 'user456',
  amount: 99.99
});

// Subscriber 1: Inventory Service
eventBus.subscribe('order.placed', async (event) => {
  await inventoryService.reserveItems(event.orderId);
});

// Subscriber 2: Email Service
eventBus.subscribe('order.placed', async (event) => {
  await emailService.sendOrderConfirmation(event.customerId);
});
```

### Event Sourcing
- Store events as the source of truth
- Rebuild state by replaying events
- Complete audit trail

```javascript
// Instead of storing current state:
// { orderId: 123, status: 'shipped', total: 99.99 }

// Store sequence of events:
// 1. OrderPlaced { orderId: 123, total: 99.99 }
// 2. OrderPaid { orderId: 123, amount: 99.99 }
// 3. OrderShipped { orderId: 123, trackingNumber: 'ABC123' }
```

### CQRS (Command Query Responsibility Segregation)
- Separate models for reading and writing
- Often used with event sourcing
- Optimized read and write operations

## Benefits

- **Loose Coupling**: Producers and consumers are independent
- **Scalability**: Easy to add new consumers
- **Flexibility**: Add new features without changing existing code
- **Asynchronous**: Non-blocking operations
- **Resilience**: Consumers can fail independently
- **Audit Trail**: Events provide complete history

## Challenges

- **Complexity**: Harder to understand flow of execution
- **Eventual Consistency**: Data might not be immediately consistent
- **Debugging**: Tracing issues across async operations
- **Event Versioning**: Managing event schema changes
- **Ordering**: Ensuring events are processed in correct order
- **Duplicate Events**: Handling idempotency

## Event Types

### Domain Events
Business-significant occurrences
```javascript
{
  type: 'OrderPlaced',
  orderId: '123',
  customerId: 'user456',
  items: [...],
  timestamp: '2024-01-15T10:30:00Z'
}
```

### Integration Events
Cross-service communication
```javascript
{
  type: 'CustomerCreatedEvent',
  customerId: '789',
  email: 'user@example.com',
  createdAt: '2024-01-15T10:30:00Z'
}
```

### Notification Events
Trigger side effects
```javascript
{
  type: 'LowStockAlert',
  productId: 'prod123',
  currentStock: 5,
  threshold: 10
}
```

## When to Use

- Systems requiring high scalability
- Microservices architectures
- Real-time data processing
- Complex workflows with multiple steps
- Systems needing audit trails
- Integration between different systems

## Example Implementation

```javascript
// Event Definition
class OrderPlacedEvent {
  constructor(orderId, customerId, items, total) {
    this.type = 'OrderPlaced';
    this.orderId = orderId;
    this.customerId = customerId;
    this.items = items;
    this.total = total;
    this.timestamp = new Date().toISOString();
  }
}

// Event Bus
class EventBus {
  constructor() {
    this.subscribers = new Map();
  }

  subscribe(eventType, handler) {
    if (!this.subscribers.has(eventType)) {
      this.subscribers.set(eventType, []);
    }
    this.subscribers.get(eventType).push(handler);
  }

  async publish(eventType, eventData) {
    const handlers = this.subscribers.get(eventType) || [];

    // Execute all handlers asynchronously
    await Promise.all(
      handlers.map(handler =>
        handler(eventData).catch(err =>
          console.error(`Handler failed: ${err}`)
        )
      )
    );
  }
}

// Usage
const eventBus = new EventBus();

// Order Service
class OrderService {
  async placeOrder(customerId, items) {
    const order = await this.createOrder(customerId, items);

    const event = new OrderPlacedEvent(
      order.id,
      customerId,
      items,
      order.total
    );

    await eventBus.publish('OrderPlaced', event);
    return order;
  }
}

// Inventory Service
eventBus.subscribe('OrderPlaced', async (event) => {
  await inventoryService.reserveItems(event.items);
});

// Email Service
eventBus.subscribe('OrderPlaced', async (event) => {
  await emailService.sendConfirmation(event.customerId, event.orderId);
});

// Analytics Service
eventBus.subscribe('OrderPlaced', async (event) => {
  await analyticsService.trackOrder(event);
});
```

## Best Practices

1. **Idempotency**: Ensure handlers can process same event multiple times
2. **Event Schema**: Use versioned schemas for events
3. **Error Handling**: Implement retry logic and dead letter queues
4. **Monitoring**: Track event flow and processing times
5. **Event Naming**: Use clear, past-tense names
6. **Small Events**: Keep event payloads focused and minimal
7. **Event Store**: Consider persisting events for replay
8. **Eventual Consistency**: Design for eventual consistency
9. **Correlation IDs**: Track related events across services
10. **Testing**: Test event handlers independently

## Related Patterns

- Saga Pattern (distributed transactions)
- Event Sourcing
- CQRS
- Publish-Subscribe
- Message Queue Pattern
