# Clean Architecture

## Overview

Clean Architecture, introduced by Robert C. Martin (Uncle Bob), is an architectural pattern that emphasizes separation of concerns through layers, with the core business logic at the center, independent of external frameworks, databases, and UI.

## Structure

```
┌─────────────────────────────────────────┐
│         Frameworks & Drivers            │  (External)
│    (Web, DB, UI, External Interfaces)   │
├─────────────────────────────────────────┤
│       Interface Adapters                │  (Controllers, Gateways,
│  (Controllers, Presenters, Gateways)    │   Presenters)
├─────────────────────────────────────────┤
│          Use Cases                      │  (Application Business
│   (Application Business Rules)          │   Rules)
├─────────────────────────────────────────┤
│          Entities                       │  (Enterprise Business
│   (Enterprise Business Rules)           │   Rules)
└─────────────────────────────────────────┘

        Dependency Direction: Inward →
```

## The Dependency Rule

**"Source code dependencies must point only inward, toward higher-level policies."**

- Inner layers know nothing about outer layers
- Business logic doesn't depend on frameworks or UI
- Entities are independent of use cases
- Use cases are independent of controllers

## Layers

### Entities (Center)
- Enterprise-wide business rules
- Most stable, least likely to change
- Pure business objects

```javascript
class Order {
  constructor(id, customer, items) {
    this.id = id;
    this.customer = customer;
    this.items = items;
  }

  calculateTotal() {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }

  applyDiscount(discountRule) {
    return discountRule.apply(this);
  }
}
```

### Use Cases
- Application-specific business rules
- Orchestrate data flow to/from entities
- Independent of UI and database

```javascript
class PlaceOrderUseCase {
  constructor(orderRepository, paymentGateway) {
    this.orderRepository = orderRepository;
    this.paymentGateway = paymentGateway;
  }

  async execute(orderData) {
    // Business logic
    const order = new Order(
      generateId(),
      orderData.customer,
      orderData.items
    );

    // Validate
    if (order.calculateTotal() <= 0) {
      throw new Error('Invalid order total');
    }

    // Process payment
    const payment = await this.paymentGateway.charge(
      orderData.customer,
      order.calculateTotal()
    );

    // Save order
    await this.orderRepository.save(order);

    return order;
  }
}
```

### Interface Adapters
- Convert data between use cases and external world
- Controllers, presenters, gateways

```javascript
// Controller (HTTP → Use Case)
class OrderController {
  constructor(placeOrderUseCase) {
    this.placeOrderUseCase = placeOrderUseCase;
  }

  async createOrder(req, res) {
    try {
      const order = await this.placeOrderUseCase.execute({
        customer: req.body.customer,
        items: req.body.items
      });

      res.status(201).json(this.presentOrder(order));
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }

  presentOrder(order) {
    return {
      id: order.id,
      customer: order.customer,
      total: order.calculateTotal(),
      items: order.items
    };
  }
}
```

### Frameworks & Drivers (Outermost)
- External frameworks and tools
- Database, web framework, UI
- Most volatile, most likely to change

## Key Principles

### Independence
- **Independent of Frameworks**: Architecture not dependent on libraries
- **Testable**: Business rules testable without UI, DB, web server
- **Independent of UI**: Swap UI without changing business rules
- **Independent of Database**: Business rules don't care about DB
- **Independent of External Agency**: Business rules isolated from outside world

### Dependency Inversion
- High-level modules don't depend on low-level modules
- Both depend on abstractions (interfaces)

```javascript
// Use case depends on abstraction, not concrete implementation
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(id: string): Promise<Order>;
}

// Concrete implementation in outer layer
class PostgresOrderRepository implements OrderRepository {
  async save(order: Order): Promise<void> {
    // PostgreSQL specific code
  }

  async findById(id: string): Promise<Order> {
    // PostgreSQL specific code
  }
}
```

## Benefits

- **Testability**: Easy to test business logic independently
- **Maintainability**: Changes in outer layers don't affect core
- **Flexibility**: Easy to swap frameworks, databases, UI
- **Framework Independence**: Not locked into specific tools
- **Business Logic Protection**: Core rules remain stable

## Challenges

- **Initial Complexity**: More boilerplate code upfront
- **Learning Curve**: Requires understanding of principles
- **Over-Engineering**: Can be overkill for simple applications
- **Indirection**: More layers mean more files and abstractions

## When to Use

- Complex business logic applications
- Long-lived systems requiring maintainability
- Systems with frequently changing UI or database
- Applications needing high testability
- Enterprise applications

## Example Structure

```
src/
├── entities/
│   ├── Order.js
│   ├── Customer.js
│   └── Product.js
├── use-cases/
│   ├── PlaceOrderUseCase.js
│   ├── GetOrderUseCase.js
│   └── CancelOrderUseCase.js
├── interface-adapters/
│   ├── controllers/
│   │   └── OrderController.js
│   ├── presenters/
│   │   └── OrderPresenter.js
│   └── repositories/
│       └── OrderRepositoryImpl.js
└── frameworks-drivers/
    ├── web/
    │   └── express-app.js
    ├── database/
    │   └── postgres-connection.js
    └── external-services/
        └── payment-gateway.js
```

## Complete Example

```javascript
// ENTITIES LAYER
class Order {
  constructor(id, items, customerId) {
    this.id = id;
    this.items = items;
    this.customerId = customerId;
    this.status = 'pending';
  }

  calculateTotal() {
    return this.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }

  confirm() {
    if (this.status !== 'pending') {
      throw new Error('Only pending orders can be confirmed');
    }
    this.status = 'confirmed';
  }
}

// USE CASES LAYER
class PlaceOrderUseCase {
  constructor(orderRepository, inventoryService, emailService) {
    this.orderRepository = orderRepository;
    this.inventoryService = inventoryService;
    this.emailService = emailService;
  }

  async execute(request) {
    // Validate inventory
    for (const item of request.items) {
      const available = await this.inventoryService.checkAvailability(item.productId);
      if (!available) {
        throw new Error(`Product ${item.productId} not available`);
      }
    }

    // Create order
    const order = new Order(generateId(), request.items, request.customerId);

    // Save order
    await this.orderRepository.save(order);

    // Send confirmation
    await this.emailService.sendOrderConfirmation(request.customerId, order.id);

    return {
      orderId: order.id,
      total: order.calculateTotal(),
      status: order.status
    };
  }
}

// INTERFACE ADAPTERS LAYER
class OrderController {
  constructor(placeOrderUseCase) {
    this.placeOrderUseCase = placeOrderUseCase;
  }

  async handlePlaceOrder(req, res) {
    try {
      const result = await this.placeOrderUseCase.execute({
        customerId: req.user.id,
        items: req.body.items
      });

      res.status(201).json({
        success: true,
        data: result
      });
    } catch (error) {
      res.status(400).json({
        success: false,
        error: error.message
      });
    }
  }
}

// FRAMEWORKS & DRIVERS LAYER
class PostgresOrderRepository {
  constructor(dbConnection) {
    this.db = dbConnection;
  }

  async save(order) {
    await this.db.query(
      'INSERT INTO orders (id, customer_id, items, status) VALUES ($1, $2, $3, $4)',
      [order.id, order.customerId, JSON.stringify(order.items), order.status]
    );
  }

  async findById(id) {
    const result = await this.db.query('SELECT * FROM orders WHERE id = $1', [id]);
    // Map database record to Order entity
    return new Order(result.id, result.items, result.customer_id);
  }
}
```

## Best Practices

1. **Follow the Dependency Rule**: Always point inward
2. **Use Interfaces**: Define interfaces in inner layers
3. **Keep Entities Pure**: No framework dependencies
4. **Test Use Cases**: Test business logic independently
5. **Thin Controllers**: Controllers should be thin adapters
6. **Dependency Injection**: Wire dependencies from outside
7. **Small Use Cases**: Each use case should do one thing
8. **Domain-Driven Design**: Combine with DDD principles

## Related Concepts

- Hexagonal Architecture (Ports & Adapters)
- Onion Architecture
- Domain-Driven Design
- SOLID Principles
- Dependency Inversion Principle
