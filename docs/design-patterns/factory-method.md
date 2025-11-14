# Factory Method Pattern

## Category
Creational Pattern

## Overview

The Factory Method pattern defines an interface for creating objects but lets subclasses decide which class to instantiate. It delegates object creation to subclasses, promoting loose coupling and adhering to the Open/Closed Principle.

## Problem

- Direct object creation (using `new`) couples code to specific classes
- Hard to extend with new types without modifying existing code
- Difficult to manage complex object creation logic
- Need flexibility in object creation without exposing creation logic

## Solution

Define an interface/method for creating objects, but let subclasses/implementations decide the actual class to instantiate. The factory method encapsulates object creation.

## Structure

```
Creator (Abstract)
├── factoryMethod() [abstract]
└── operation()
    └── calls factoryMethod()

ConcreteCreatorA           ConcreteCreatorB
└── factoryMethod()        └── factoryMethod()
    └── returns ProductA       └── returns ProductB
```

## Implementation

### Basic Factory Method

```javascript
// Product interface
class Vehicle {
  deliver() {
    throw new Error('deliver() must be implemented');
  }
}

// Concrete products
class Truck extends Vehicle {
  deliver() {
    return 'Delivering by land in a truck';
  }
}

class Ship extends Vehicle {
  deliver() {
    return 'Delivering by sea in a ship';
  }
}

class Plane extends Vehicle {
  deliver() {
    return 'Delivering by air in a plane';
  }
}

// Creator (Factory)
class Logistics {
  // Factory method - to be overridden
  createTransport() {
    throw new Error('createTransport() must be implemented');
  }

  // Business logic using the factory method
  planDelivery() {
    const transport = this.createTransport();
    return transport.deliver();
  }
}

// Concrete creators
class RoadLogistics extends Logistics {
  createTransport() {
    return new Truck();
  }
}

class SeaLogistics extends Logistics {
  createTransport() {
    return new Ship();
  }
}

class AirLogistics extends Logistics {
  createTransport() {
    return new Plane();
  }
}

// Usage
const roadLogistics = new RoadLogistics();
console.log(roadLogistics.planDelivery());
// "Delivering by land in a truck"

const seaLogistics = new SeaLogistics();
console.log(seaLogistics.planDelivery());
// "Delivering by sea in a ship"
```

### Simple Factory (Factory Pattern)

Not technically Factory Method, but commonly used:

```javascript
class VehicleFactory {
  static createVehicle(type) {
    switch (type) {
      case 'truck':
        return new Truck();
      case 'ship':
        return new Ship();
      case 'plane':
        return new Plane();
      default:
        throw new Error(`Unknown vehicle type: ${type}`);
    }
  }
}

// Usage
const truck = VehicleFactory.createVehicle('truck');
console.log(truck.deliver());

const ship = VehicleFactory.createVehicle('ship');
console.log(ship.deliver());
```

### Factory Method with Parameters

```javascript
// Products
class Button {
  render() {
    throw new Error('render() must be implemented');
  }
}

class WindowsButton extends Button {
  render() {
    return '<button class="windows">Windows Button</button>';
  }
}

class MacButton extends Button {
  render() {
    return '<button class="mac">Mac Button</button>';
  }
}

class LinuxButton extends Button {
  render() {
    return '<button class="linux">Linux Button</button>';
  }
}

// Factory
class Dialog {
  createButton() {
    throw new Error('createButton() must be implemented');
  }

  render() {
    const button = this.createButton();
    return `<div class="dialog">${button.render()}</div>`;
  }
}

// Concrete factories
class WindowsDialog extends Dialog {
  createButton() {
    return new WindowsButton();
  }
}

class MacDialog extends Dialog {
  createButton() {
    return new MacButton();
  }
}

class LinuxDialog extends Dialog {
  createButton() {
    return new LinuxButton();
  }
}

// Factory selection based on environment
function getDialog(platform) {
  switch (platform) {
    case 'windows':
      return new WindowsDialog();
    case 'mac':
      return new MacDialog();
    case 'linux':
      return new LinuxDialog();
    default:
      return new WindowsDialog();
  }
}

// Usage
const platform = 'mac';
const dialog = getDialog(platform);
console.log(dialog.render());
```

## Real-World Examples

### Document Creator

```javascript
// Product
class Document {
  constructor() {
    this.pages = [];
  }

  addPage(content) {
    this.pages.push(content);
  }

  render() {
    throw new Error('render() must be implemented');
  }
}

// Concrete products
class PDFDocument extends Document {
  render() {
    return {
      type: 'PDF',
      content: this.pages.join('\n'),
      metadata: { format: 'PDF', version: '1.7' }
    };
  }
}

class WordDocument extends Document {
  render() {
    return {
      type: 'DOCX',
      content: this.pages.join('\n\n'),
      metadata: { format: 'DOCX', version: '2019' }
    };
  }
}

class HTMLDocument extends Document {
  render() {
    return {
      type: 'HTML',
      content: `<html><body>${this.pages.join('<hr>')}</body></html>`,
      metadata: { format: 'HTML5' }
    };
  }
}

// Factory
class DocumentCreator {
  createDocument() {
    throw new Error('createDocument() must be implemented');
  }

  newDocument(pages) {
    const doc = this.createDocument();
    pages.forEach(page => doc.addPage(page));
    return doc.render();
  }
}

// Concrete factories
class PDFCreator extends DocumentCreator {
  createDocument() {
    return new PDFDocument();
  }
}

class WordCreator extends DocumentCreator {
  createDocument() {
    return new WordDocument();
  }
}

class HTMLCreator extends DocumentCreator {
  createDocument() {
    return new HTMLDocument();
  }
}

// Usage
const pages = ['Page 1 content', 'Page 2 content', 'Page 3 content'];

const pdfCreator = new PDFCreator();
const pdf = pdfCreator.newDocument(pages);
console.log(pdf);

const wordCreator = new WordCreator();
const word = wordCreator.newDocument(pages);
console.log(word);
```

### Database Connection Factory

```javascript
// Product
class DatabaseConnection {
  connect() {
    throw new Error('connect() must be implemented');
  }

  query(sql) {
    throw new Error('query() must be implemented');
  }

  disconnect() {
    throw new Error('disconnect() must be implemented');
  }
}

// Concrete products
class MySQLConnection extends DatabaseConnection {
  constructor(config) {
    super();
    this.config = config;
    this.connected = false;
  }

  connect() {
    console.log(`Connecting to MySQL: ${this.config.host}`);
    this.connected = true;
    return this;
  }

  query(sql) {
    if (!this.connected) throw new Error('Not connected');
    console.log(`MySQL Query: ${sql}`);
    return { rows: [] };
  }

  disconnect() {
    console.log('Disconnecting from MySQL');
    this.connected = false;
  }
}

class PostgreSQLConnection extends DatabaseConnection {
  constructor(config) {
    super();
    this.config = config;
    this.connected = false;
  }

  connect() {
    console.log(`Connecting to PostgreSQL: ${this.config.host}`);
    this.connected = true;
    return this;
  }

  query(sql) {
    if (!this.connected) throw new Error('Not connected');
    console.log(`PostgreSQL Query: ${sql}`);
    return { rows: [] };
  }

  disconnect() {
    console.log('Disconnecting from PostgreSQL');
    this.connected = false;
  }
}

class MongoDBConnection extends DatabaseConnection {
  constructor(config) {
    super();
    this.config = config;
    this.connected = false;
  }

  connect() {
    console.log(`Connecting to MongoDB: ${this.config.host}`);
    this.connected = true;
    return this;
  }

  query(filter) {
    if (!this.connected) throw new Error('Not connected');
    console.log(`MongoDB Query: ${JSON.stringify(filter)}`);
    return { documents: [] };
  }

  disconnect() {
    console.log('Disconnecting from MongoDB');
    this.connected = false;
  }
}

// Factory
class DatabaseFactory {
  static createConnection(type, config) {
    switch (type) {
      case 'mysql':
        return new MySQLConnection(config);
      case 'postgresql':
        return new PostgreSQLConnection(config);
      case 'mongodb':
        return new MongoDBConnection(config);
      default:
        throw new Error(`Unsupported database type: ${type}`);
    }
  }
}

// Usage
const config = { host: 'localhost', port: 5432, database: 'myapp' };

const dbType = process.env.DB_TYPE || 'postgresql';
const connection = DatabaseFactory.createConnection(dbType, config);

connection.connect();
connection.query('SELECT * FROM users');
connection.disconnect();
```

### Payment Processor Factory

```javascript
// Product
class PaymentProcessor {
  processPayment(amount) {
    throw new Error('processPayment() must be implemented');
  }

  refund(transactionId) {
    throw new Error('refund() must be implemented');
  }
}

// Concrete products
class CreditCardProcessor extends PaymentProcessor {
  processPayment(amount) {
    console.log(`Processing credit card payment: $${amount}`);
    return {
      success: true,
      transactionId: `CC-${Date.now()}`,
      method: 'credit_card',
      amount
    };
  }

  refund(transactionId) {
    console.log(`Refunding credit card transaction: ${transactionId}`);
    return { success: true, refunded: true };
  }
}

class PayPalProcessor extends PaymentProcessor {
  processPayment(amount) {
    console.log(`Processing PayPal payment: $${amount}`);
    return {
      success: true,
      transactionId: `PP-${Date.now()}`,
      method: 'paypal',
      amount
    };
  }

  refund(transactionId) {
    console.log(`Refunding PayPal transaction: ${transactionId}`);
    return { success: true, refunded: true };
  }
}

class CryptoProcessor extends PaymentProcessor {
  processPayment(amount) {
    console.log(`Processing cryptocurrency payment: $${amount}`);
    return {
      success: true,
      transactionId: `CRYPTO-${Date.now()}`,
      method: 'cryptocurrency',
      amount
    };
  }

  refund(transactionId) {
    console.log(`Refunding crypto transaction: ${transactionId}`);
    return { success: true, refunded: true };
  }
}

// Factory
class PaymentFactory {
  static createProcessor(method) {
    switch (method) {
      case 'credit_card':
        return new CreditCardProcessor();
      case 'paypal':
        return new PayPalProcessor();
      case 'crypto':
        return new CryptoProcessor();
      default:
        throw new Error(`Unsupported payment method: ${method}`);
    }
  }
}

// Usage
class CheckoutService {
  processOrder(paymentMethod, amount) {
    const processor = PaymentFactory.createProcessor(paymentMethod);
    return processor.processPayment(amount);
  }
}

const checkout = new CheckoutService();
const result = checkout.processOrder('paypal', 99.99);
console.log(result);
```

## Benefits

- **Loose Coupling**: Code depends on abstractions, not concrete classes
- **Open/Closed Principle**: Easy to add new types without modifying existing code
- **Single Responsibility**: Object creation logic in one place
- **Flexibility**: Easy to extend with new product types
- **Testability**: Easy to mock factories and products

## Drawbacks

- **Complexity**: More classes and interfaces
- **Indirection**: Extra layer between client and products
- **Overhead**: Might be overkill for simple cases

## When to Use

✅ **Good Use Cases**:
- Multiple related classes with similar behavior
- Object creation is complex
- Need to decouple creation from usage
- Product types determined at runtime
- Need to extend with new types frequently

❌ **Avoid When**:
- Only one product type
- Creation logic is trivial
- No need for extensibility

## Comparison with Other Patterns

### Factory Method vs Abstract Factory
- **Factory Method**: Single product creation
- **Abstract Factory**: Family of related products

### Factory Method vs Builder
- **Factory Method**: Creates complete objects in one step
- **Builder**: Constructs complex objects step by step

### Factory Method vs Prototype
- **Factory Method**: Creates new instances from scratch
- **Prototype**: Creates new instances by cloning

## Best Practices

1. **Return Interfaces**: Factory should return abstract types
2. **Naming**: Use clear names (create*, make*, build*)
3. **Parameters**: Use parameters to determine type if needed
4. **Error Handling**: Handle unknown types gracefully
5. **Registration**: Consider allowing dynamic registration
6. **Configuration**: Use config for factory selection

### Dynamic Registration Example

```javascript
class VehicleRegistry {
  constructor() {
    this.vehicles = new Map();
  }

  register(type, vehicleClass) {
    this.vehicles.set(type, vehicleClass);
  }

  create(type, ...args) {
    const VehicleClass = this.vehicles.get(type);
    if (!VehicleClass) {
      throw new Error(`Unknown vehicle type: ${type}`);
    }
    return new VehicleClass(...args);
  }
}

// Usage
const registry = new VehicleRegistry();
registry.register('truck', Truck);
registry.register('ship', Ship);
registry.register('plane', Plane);

const vehicle = registry.create('truck');
```

## Related Patterns

- **Abstract Factory**: Creates families of related objects
- **Prototype**: Alternative creational pattern
- **Singleton**: Factory can be a singleton
- **Template Method**: Similar structure, different purpose

## Common Mistakes

```javascript
// ❌ Bad: Violates Open/Closed
class BadFactory {
  create(type) {
    if (type === 'A') return new ProductA();
    if (type === 'B') return new ProductB();
    // Have to modify for each new type
  }
}

// ✅ Good: Extensible
class GoodFactory {
  constructor() {
    this.types = new Map();
  }

  register(type, creator) {
    this.types.set(type, creator);
  }

  create(type) {
    const creator = this.types.get(type);
    if (!creator) throw new Error(`Unknown type: ${type}`);
    return creator();
  }
}

// ❌ Bad: Tight coupling
const product = new ConcreteProduct(); // Direct instantiation

// ✅ Good: Factory decouples
const product = factory.create('product');
```
