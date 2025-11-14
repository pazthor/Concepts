# Hexagonal Architecture (Ports and Adapters)

## Overview

Hexagonal Architecture, also known as Ports and Adapters, was introduced by Alistair Cockburn. It creates a clear separation between the core application logic and external concerns, making the application independent of frameworks, databases, and UI.

## Structure

```
         ┌─────────────────────────────┐
         │     External World          │
         │  (UI, DB, APIs, Services)   │
         └──────────┬──────────────────┘
                    │
         ┌──────────▼──────────────────┐
         │       Adapters              │  (Framework-specific)
         │  (REST, GraphQL, SQL, etc)  │
         └──────────┬──────────────────┘
                    │
         ┌──────────▼──────────────────┐
         │        Ports                │  (Interfaces)
         │   (Contracts/Interfaces)    │
         └──────────┬──────────────────┘
                    │
         ┌──────────▼──────────────────┐
         │    Application Core         │  (Business Logic)
         │   (Domain + Use Cases)      │
         └─────────────────────────────┘
```

## Key Concepts

### The Hexagon (Application Core)
- Contains business logic and domain model
- Technology-agnostic
- No dependencies on external frameworks

### Ports
- Interfaces that define how to interact with the application
- Two types: **Primary (Driving)** and **Secondary (Driven)**

#### Primary Ports (Driving)
- Define what the application can do
- Used by actors to drive the application
- Examples: Use case interfaces, service interfaces

#### Secondary Ports (Driven)
- Define what the application needs
- Required by application to function
- Examples: Repository interfaces, external service interfaces

### Adapters
- Concrete implementations of ports
- Convert external requests to application calls
- Two types: **Primary (Driving)** and **Secondary (Driven)**

#### Primary Adapters (Driving)
- Initiate interactions with the application
- Examples: REST controllers, GraphQL resolvers, CLI commands

#### Secondary Adapters (Driven)
- Provide implementations for application needs
- Examples: Database repositories, external API clients

## Ports and Adapters Flow

```
User → REST Adapter → Primary Port → Application Core → Secondary Port → Database Adapter → Database
```

## Benefits

- **Testability**: Easy to test with mock adapters
- **Flexibility**: Swap adapters without changing core
- **Technology Independence**: Core independent of frameworks
- **Multiple Interfaces**: Support multiple UIs simultaneously
- **Deferral of Technology Decisions**: Choose DB/framework later
- **Business Logic Protection**: Core remains clean

## Example Implementation

### Application Core

```javascript
// Domain Entity
class Order {
  constructor(id, customerId, items) {
    this.id = id;
    this.customerId = customerId;
    this.items = items;
    this.status = 'pending';
  }

  calculateTotal() {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }

  confirm() {
    this.status = 'confirmed';
  }
}

// Use Case
class CreateOrderUseCase {
  constructor(orderRepository, emailService) {
    // Dependencies are port interfaces
    this.orderRepository = orderRepository;
    this.emailService = emailService;
  }

  async execute(customerId, items) {
    // Business logic
    const order = new Order(generateId(), customerId, items);

    if (order.calculateTotal() <= 0) {
      throw new Error('Order total must be greater than zero');
    }

    // Use secondary ports
    await this.orderRepository.save(order);
    await this.emailService.sendConfirmation(customerId, order.id);

    return order;
  }
}
```

### Ports (Interfaces)

```typescript
// Primary Port (Driving)
// Defines what the application can do
interface OrderService {
  createOrder(customerId: string, items: OrderItem[]): Promise<Order>;
  getOrder(orderId: string): Promise<Order>;
  cancelOrder(orderId: string): Promise<void>;
}

// Secondary Ports (Driven)
// Defines what the application needs
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order>;
  findByCustomerId(customerId: string): Promise<Order[]>;
}

interface EmailService {
  sendConfirmation(customerId: string, orderId: string): Promise<void>;
}
```

### Primary Adapters (Driving)

```javascript
// REST Adapter
class RestOrderController {
  constructor(orderService) {
    // orderService implements OrderService port
    this.orderService = orderService;
  }

  async createOrder(req, res) {
    try {
      const order = await this.orderService.createOrder(
        req.body.customerId,
        req.body.items
      );

      res.status(201).json({
        id: order.id,
        total: order.calculateTotal(),
        status: order.status
      });
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}

// GraphQL Adapter
class GraphQLOrderResolver {
  constructor(orderService) {
    this.orderService = orderService;
  }

  async createOrder(parent, args, context) {
    return this.orderService.createOrder(
      args.customerId,
      args.items
    );
  }
}

// CLI Adapter
class CLIOrderHandler {
  constructor(orderService) {
    this.orderService = orderService;
  }

  async handleCreateOrderCommand(customerId, items) {
    const order = await this.orderService.createOrder(customerId, items);
    console.log(`Order created: ${order.id}`);
  }
}
```

### Secondary Adapters (Driven)

```javascript
// PostgreSQL Adapter
class PostgresOrderRepository {
  constructor(dbConnection) {
    this.db = dbConnection;
  }

  async save(order) {
    await this.db.query(
      `INSERT INTO orders (id, customer_id, items, status, total)
       VALUES ($1, $2, $3, $4, $5)`,
      [
        order.id,
        order.customerId,
        JSON.stringify(order.items),
        order.status,
        order.calculateTotal()
      ]
    );
  }

  async findById(id) {
    const result = await this.db.query(
      'SELECT * FROM orders WHERE id = $1',
      [id]
    );
    return this.mapToOrder(result.rows[0]);
  }

  mapToOrder(row) {
    return new Order(row.id, row.customer_id, JSON.parse(row.items));
  }
}

// MongoDB Adapter
class MongoOrderRepository {
  constructor(mongoClient) {
    this.client = mongoClient;
    this.collection = this.client.db('shop').collection('orders');
  }

  async save(order) {
    await this.collection.insertOne({
      _id: order.id,
      customerId: order.customerId,
      items: order.items,
      status: order.status,
      total: order.calculateTotal()
    });
  }

  async findById(id) {
    const doc = await this.collection.findOne({ _id: id });
    return new Order(doc._id, doc.customerId, doc.items);
  }
}

// Email Service Adapter
class SendGridEmailAdapter {
  constructor(sendGridClient) {
    this.client = sendGridClient;
  }

  async sendConfirmation(customerId, orderId) {
    await this.client.send({
      to: await this.getCustomerEmail(customerId),
      subject: 'Order Confirmation',
      text: `Your order ${orderId} has been confirmed`
    });
  }
}
```

## Dependency Injection Setup

```javascript
// Wire everything together
class Application {
  static create(config) {
    // Secondary adapters
    const dbConnection = createDbConnection(config.database);
    const orderRepository = new PostgresOrderRepository(dbConnection);
    const emailService = new SendGridEmailAdapter(config.sendGrid);

    // Application core
    const createOrderUseCase = new CreateOrderUseCase(
      orderRepository,
      emailService
    );

    // Primary adapters
    const restController = new RestOrderController(createOrderUseCase);
    const graphqlResolver = new GraphQLOrderResolver(createOrderUseCase);

    return {
      restController,
      graphqlResolver
    };
  }
}

// Usage
const app = Application.create(config);
```

## Testing

### Testing with Mock Adapters

```javascript
// Mock repository (test double)
class InMemoryOrderRepository {
  constructor() {
    this.orders = new Map();
  }

  async save(order) {
    this.orders.set(order.id, order);
  }

  async findById(id) {
    return this.orders.get(id);
  }
}

// Mock email service
class MockEmailService {
  constructor() {
    this.sentEmails = [];
  }

  async sendConfirmation(customerId, orderId) {
    this.sentEmails.push({ customerId, orderId });
  }
}

// Test
describe('CreateOrderUseCase', () => {
  it('should create order and send confirmation', async () => {
    // Arrange
    const orderRepository = new InMemoryOrderRepository();
    const emailService = new MockEmailService();
    const useCase = new CreateOrderUseCase(orderRepository, emailService);

    // Act
    const order = await useCase.execute('customer123', [
      { productId: 'p1', price: 10 }
    ]);

    // Assert
    expect(order.id).toBeDefined();
    expect(order.status).toBe('pending');
    expect(emailService.sentEmails).toHaveLength(1);
    expect(emailService.sentEmails[0].orderId).toBe(order.id);
  });
});
```

## When to Use

- Applications with complex business logic
- Systems that may need multiple interfaces (REST, GraphQL, CLI)
- Projects where database choice may change
- Applications requiring high testability
- Long-lived systems needing maintainability

## Best Practices

1. **Define Clear Ports**: Make interfaces explicit and focused
2. **Keep Core Pure**: No framework dependencies in core
3. **Test Through Ports**: Test use cases with mock adapters
4. **One Adapter Per Technology**: Each tech gets its own adapter
5. **Dependency Injection**: Wire dependencies from outside
6. **Interface Segregation**: Small, focused port interfaces
7. **Domain-Driven Design**: Combine with DDD principles
8. **Configuration at Edge**: Keep config in adapters

## Comparison with Clean Architecture

| Aspect | Hexagonal | Clean Architecture |
|--------|-----------|-------------------|
| Focus | Ports & Adapters separation | Concentric layers |
| Ports | Explicit interfaces | Implied boundaries |
| Layers | 2 (Core + Adapters) | 4 (Entities, Use Cases, Adapters, Frameworks) |
| Terminology | Ports, Adapters | Layers, Boundaries |

Both achieve similar goals with slightly different approaches.

## Related Patterns

- Clean Architecture
- Onion Architecture
- Domain-Driven Design
- Dependency Inversion Principle
- Adapter Pattern
