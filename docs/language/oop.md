# Object-Oriented Programming (OOP)

## Overview

Object-Oriented Programming is a programming paradigm based on the concept of "objects" that contain data (attributes) and code (methods). It organizes software design around data and objects rather than functions and logic.

## Core Principles

### 1. Encapsulation

**Definition**: Bundling data and methods that operate on that data within a single unit (class), and restricting access to internal details.

```javascript
class BankAccount {
  #balance; // Private field

  constructor(initialBalance) {
    this.#balance = initialBalance;
  }

  deposit(amount) {
    if (amount > 0) {
      this.#balance += amount;
    }
  }

  withdraw(amount) {
    if (amount > 0 && amount <= this.#balance) {
      this.#balance -= amount;
      return true;
    }
    return false;
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount(1000);
account.deposit(500);
console.log(account.getBalance()); // 1500
// account.#balance; // Error: Private field
```

**Benefits**:
- Data hiding and protection
- Controlled access through public methods
- Internal implementation can change without affecting external code
- Reduces complexity

### 2. Abstraction

**Definition**: Hiding complex implementation details and showing only essential features of an object.

```javascript
// Abstract class (conceptual - JavaScript doesn't have native abstract classes)
class PaymentProcessor {
  processPayment(amount) {
    throw new Error('Must implement processPayment method');
  }

  refund(transactionId) {
    throw new Error('Must implement refund method');
  }
}

class CreditCardProcessor extends PaymentProcessor {
  processPayment(amount) {
    // Complex credit card processing logic hidden
    this.validateCard();
    this.checkFraud();
    this.authorizeTransaction();
    this.capturePayment(amount);
    return { success: true, transactionId: '12345' };
  }

  refund(transactionId) {
    // Complex refund logic hidden
    return this.processRefund(transactionId);
  }

  // Private implementation details
  validateCard() { /* ... */ }
  checkFraud() { /* ... */ }
  authorizeTransaction() { /* ... */ }
  capturePayment(amount) { /* ... */ }
  processRefund(id) { /* ... */ }
}

// User doesn't need to know implementation details
const processor = new CreditCardProcessor();
processor.processPayment(100); // Simple interface
```

**Benefits**:
- Simplifies complex systems
- Reduces code duplication
- Makes code more maintainable
- Allows focusing on high-level functionality

### 3. Inheritance

**Definition**: Mechanism where a new class derives properties and methods from an existing class.

```javascript
// Base class
class Animal {
  constructor(name) {
    this.name = name;
  }

  makeSound() {
    console.log('Some generic sound');
  }

  sleep() {
    console.log(`${this.name} is sleeping`);
  }
}

// Derived class
class Dog extends Animal {
  constructor(name, breed) {
    super(name); // Call parent constructor
    this.breed = breed;
  }

  makeSound() {
    console.log('Woof! Woof!');
  }

  fetch() {
    console.log(`${this.name} is fetching the ball`);
  }
}

const dog = new Dog('Buddy', 'Golden Retriever');
dog.makeSound(); // Woof! Woof! (overridden)
dog.sleep();     // Buddy is sleeping (inherited)
dog.fetch();     // Buddy is fetching the ball (own method)
```

**Types of Inheritance**:
- **Single Inheritance**: Class inherits from one parent
- **Multiple Inheritance**: Class inherits from multiple parents (not in JavaScript)
- **Multilevel Inheritance**: Chain of inheritance (A → B → C)
- **Hierarchical Inheritance**: Multiple classes inherit from one parent

**Benefits**:
- Code reusability
- Establishes relationships between classes
- Supports polymorphism

**Caution**:
- Prefer composition over inheritance for flexibility
- Deep inheritance hierarchies can be problematic

### 4. Polymorphism

**Definition**: Ability of objects to take multiple forms, allowing different classes to be treated through the same interface.

#### Method Overriding (Runtime Polymorphism)

```javascript
class Shape {
  calculateArea() {
    throw new Error('Must implement calculateArea');
  }

  describe() {
    return `This shape has an area of ${this.calculateArea()}`;
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  calculateArea() {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  calculateArea() {
    return this.width * this.height;
  }
}

// Polymorphic behavior
function printShapeArea(shape) {
  console.log(shape.describe());
}

printShapeArea(new Circle(5));     // Works with Circle
printShapeArea(new Rectangle(4, 6)); // Works with Rectangle
```

#### Method Overloading (Compile-time Polymorphism)

JavaScript doesn't support traditional method overloading, but similar behavior can be achieved:

```javascript
class Calculator {
  add(...numbers) {
    return numbers.reduce((sum, num) => sum + num, 0);
  }
}

const calc = new Calculator();
console.log(calc.add(2, 3));        // 5
console.log(calc.add(2, 3, 4));     // 9
console.log(calc.add(1, 2, 3, 4));  // 10
```

**Benefits**:
- Flexibility in code
- Extensibility
- Cleaner, more maintainable code

## Additional OOP Concepts

### Classes and Objects

```javascript
// Class: Blueprint for objects
class Car {
  constructor(make, model, year) {
    this.make = make;
    this.model = model;
    this.year = year;
    this.mileage = 0;
  }

  drive(miles) {
    this.mileage += miles;
  }

  getInfo() {
    return `${this.year} ${this.make} ${this.model}`;
  }
}

// Objects: Instances of class
const car1 = new Car('Toyota', 'Camry', 2020);
const car2 = new Car('Honda', 'Civic', 2021);
```

### Static Members

```javascript
class MathUtils {
  static PI = 3.14159;

  static square(x) {
    return x * x;
  }

  static cube(x) {
    return x * x * x;
  }
}

console.log(MathUtils.PI);        // 3.14159
console.log(MathUtils.square(5)); // 25
// No need to create instance
```

### Composition (Alternative to Inheritance)

```javascript
// Prefer composition over inheritance
class Engine {
  start() {
    console.log('Engine starting...');
  }

  stop() {
    console.log('Engine stopping...');
  }
}

class GPS {
  navigate(destination) {
    console.log(`Navigating to ${destination}`);
  }
}

class Car {
  constructor() {
    this.engine = new Engine();    // Has-a relationship
    this.gps = new GPS();          // Has-a relationship
  }

  start() {
    this.engine.start();
  }

  navigateTo(destination) {
    this.gps.navigate(destination);
  }
}

const car = new Car();
car.start();
car.navigateTo('New York');
```

## SOLID Principles

### S - Single Responsibility Principle
A class should have only one reason to change.

```javascript
// Bad: Multiple responsibilities
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  saveToDatabase() { /* ... */ }
  sendEmail() { /* ... */ }
  generateReport() { /* ... */ }
}

// Good: Single responsibility
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

class UserRepository {
  save(user) { /* ... */ }
}

class EmailService {
  send(user, message) { /* ... */ }
}

class ReportGenerator {
  generate(user) { /* ... */ }
}
```

### O - Open/Closed Principle
Open for extension, closed for modification.

```javascript
// Bad: Need to modify class to add new shapes
class AreaCalculator {
  calculate(shape) {
    if (shape.type === 'circle') {
      return Math.PI * shape.radius ** 2;
    } else if (shape.type === 'rectangle') {
      return shape.width * shape.height;
    }
    // Need to modify when adding new shape
  }
}

// Good: Extend without modifying
class Shape {
  calculateArea() {
    throw new Error('Must implement');
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  calculateArea() {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  calculateArea() {
    return this.width * this.height;
  }
}

// No modification needed for new shapes
```

### L - Liskov Substitution Principle
Subtypes must be substitutable for their base types.

```javascript
// Bad: Violates LSP
class Bird {
  fly() {
    console.log('Flying');
  }
}

class Penguin extends Bird {
  fly() {
    throw new Error('Penguins cannot fly!');
  }
}

// Good: Respects LSP
class Bird {
  move() {
    console.log('Moving');
  }
}

class FlyingBird extends Bird {
  move() {
    console.log('Flying');
  }
}

class Penguin extends Bird {
  move() {
    console.log('Swimming');
  }
}
```

### I - Interface Segregation Principle
Many specific interfaces are better than one general interface.

```javascript
// Bad: Fat interface
class Worker {
  work() { }
  eat() { }
  sleep() { }
}

// Good: Segregated interfaces
class Workable {
  work() { }
}

class Eatable {
  eat() { }
}

class Human {
  constructor() {
    this.workable = new Workable();
    this.eatable = new Eatable();
  }
}

class Robot {
  constructor() {
    this.workable = new Workable();
    // Robots don't eat
  }
}
```

### D - Dependency Inversion Principle
Depend on abstractions, not concretions.

```javascript
// Bad: High-level depends on low-level
class MySQLDatabase {
  save(data) { /* MySQL specific */ }
}

class UserService {
  constructor() {
    this.database = new MySQLDatabase(); // Tight coupling
  }
}

// Good: Both depend on abstraction
class Database {
  save(data) {
    throw new Error('Must implement');
  }
}

class MySQLDatabase extends Database {
  save(data) { /* MySQL specific */ }
}

class MongoDatabase extends Database {
  save(data) { /* Mongo specific */ }
}

class UserService {
  constructor(database) {
    this.database = database; // Depends on abstraction
  }
}

// Flexible instantiation
const service = new UserService(new MySQLDatabase());
```

## Benefits of OOP

- **Modularity**: Code is organized into self-contained objects
- **Reusability**: Code can be reused through inheritance and composition
- **Maintainability**: Easier to modify and maintain
- **Security**: Data hiding through encapsulation
- **Problem Solving**: Models real-world entities naturally

## Drawbacks of OOP

- **Complexity**: Can be over-engineered for simple problems
- **Performance**: Abstraction layers can impact performance
- **Learning Curve**: Requires understanding of concepts
- **Not Always Natural**: Some problems better suited for other paradigms

## When to Use OOP

- Complex applications with many entities
- Systems modeling real-world objects
- Projects requiring code reusability
- Large teams needing modular code
- Applications with changing requirements

## Best Practices

1. **Favor Composition Over Inheritance**: More flexible
2. **Program to Interfaces**: Depend on abstractions
3. **Keep Classes Small**: Single Responsibility
4. **Use Encapsulation**: Hide implementation details
5. **Follow SOLID Principles**: Better design
6. **Avoid Deep Inheritance**: Prefer shallow hierarchies
7. **Use Polymorphism**: Write flexible code
8. **Name Classes Well**: Clear, descriptive names
