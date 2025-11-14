# Strategy Pattern

## Category
Behavioral Pattern

## Overview

The Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it. It enables selecting an algorithm at runtime.

## Problem

- Different variants of an algorithm needed
- Want to switch algorithms at runtime
- Many conditional statements to select behavior
- Algorithm implementation details exposed to client
- Hard to add new algorithms without modifying existing code

## Solution

Extract algorithms into separate classes with a common interface. The context uses this interface to call the algorithm, allowing runtime selection of which strategy to use.

## Structure

```
Context
├── strategy: Strategy
└── executeStrategy()

Strategy (Interface)
└── execute()

ConcreteStrategyA    ConcreteStrategyB    ConcreteStrategyC
└── execute()        └── execute()        └── execute()
```

## Implementation

### Basic Strategy

```javascript
// Strategy interface
class PaymentStrategy {
  pay(amount) {
    throw new Error('pay() must be implemented');
  }
}

// Concrete strategies
class CreditCardStrategy extends PaymentStrategy {
  constructor(cardNumber, cvv) {
    super();
    this.cardNumber = cardNumber;
    this.cvv = cvv;
  }

  pay(amount) {
    console.log(`Paid $${amount} with credit card ${this.cardNumber}`);
    return { success: true, method: 'credit_card', amount };
  }
}

class PayPalStrategy extends PaymentStrategy {
  constructor(email) {
    super();
    this.email = email;
  }

  pay(amount) {
    console.log(`Paid $${amount} via PayPal account ${this.email}`);
    return { success: true, method: 'paypal', amount };
  }
}

class BitcoinStrategy extends PaymentStrategy {
  constructor(walletAddress) {
    super();
    this.walletAddress = walletAddress;
  }

  pay(amount) {
    console.log(`Paid $${amount} via Bitcoin to ${this.walletAddress}`);
    return { success: true, method: 'bitcoin', amount };
  }
}

// Context
class ShoppingCart {
  constructor() {
    this.items = [];
    this.paymentStrategy = null;
  }

  addItem(item) {
    this.items.push(item);
  }

  setPaymentStrategy(strategy) {
    this.paymentStrategy = strategy;
  }

  calculateTotal() {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }

  checkout() {
    if (!this.paymentStrategy) {
      throw new Error('Payment strategy not set');
    }

    const total = this.calculateTotal();
    return this.paymentStrategy.pay(total);
  }
}

// Usage
const cart = new ShoppingCart();
cart.addItem({ name: 'Book', price: 15 });
cart.addItem({ name: 'Pen', price: 5 });

// Use credit card
cart.setPaymentStrategy(new CreditCardStrategy('1234-5678', '123'));
cart.checkout(); // Paid $20 with credit card

// Switch to PayPal
cart.setPaymentStrategy(new PayPalStrategy('user@example.com'));
cart.checkout(); // Paid $20 via PayPal
```

### Function-Based Strategy (JavaScript)

```javascript
// Strategies as functions
const strategies = {
  creditCard: (amount, details) => {
    console.log(`Paid $${amount} with card ${details.cardNumber}`);
    return { success: true, method: 'credit_card' };
  },

  paypal: (amount, details) => {
    console.log(`Paid $${amount} via PayPal ${details.email}`);
    return { success: true, method: 'paypal' };
  },

  bitcoin: (amount, details) => {
    console.log(`Paid $${amount} via Bitcoin ${details.wallet}`);
    return { success: true, method: 'bitcoin' };
  }
};

// Context
class PaymentProcessor {
  constructor() {
    this.strategy = null;
    this.details = null;
  }

  setStrategy(strategyName, details) {
    this.strategy = strategies[strategyName];
    this.details = details;
  }

  process(amount) {
    if (!this.strategy) {
      throw new Error('Strategy not set');
    }
    return this.strategy(amount, this.details);
  }
}

// Usage
const processor = new PaymentProcessor();
processor.setStrategy('paypal', { email: 'user@example.com' });
processor.process(100);
```

## Real-World Examples

### Sorting Strategies

```javascript
// Sorting strategies
class SortStrategy {
  sort(data) {
    throw new Error('sort() must be implemented');
  }
}

class BubbleSortStrategy extends SortStrategy {
  sort(data) {
    console.log('Using Bubble Sort');
    const arr = [...data];
    const n = arr.length;

    for (let i = 0; i < n - 1; i++) {
      for (let j = 0; j < n - i - 1; j++) {
        if (arr[j] > arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    return arr;
  }
}

class QuickSortStrategy extends SortStrategy {
  sort(data) {
    console.log('Using Quick Sort');
    return this.quickSort([...data]);
  }

  quickSort(arr) {
    if (arr.length <= 1) return arr;

    const pivot = arr[Math.floor(arr.length / 2)];
    const left = arr.filter(x => x < pivot);
    const middle = arr.filter(x => x === pivot);
    const right = arr.filter(x => x > pivot);

    return [...this.quickSort(left), ...middle, ...this.quickSort(right)];
  }
}

class MergeSortStrategy extends SortStrategy {
  sort(data) {
    console.log('Using Merge Sort');
    return this.mergeSort([...data]);
  }

  mergeSort(arr) {
    if (arr.length <= 1) return arr;

    const mid = Math.floor(arr.length / 2);
    const left = this.mergeSort(arr.slice(0, mid));
    const right = this.mergeSort(arr.slice(mid));

    return this.merge(left, right);
  }

  merge(left, right) {
    const result = [];
    let i = 0, j = 0;

    while (i < left.length && j < right.length) {
      if (left[i] < right[j]) {
        result.push(left[i++]);
      } else {
        result.push(right[j++]);
      }
    }

    return result.concat(left.slice(i)).concat(right.slice(j));
  }
}

// Context
class DataSorter {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  sort(data) {
    if (data.length < 10) {
      // Use bubble sort for small arrays
      this.setStrategy(new BubbleSortStrategy());
    } else if (data.length < 1000) {
      // Use quick sort for medium arrays
      this.setStrategy(new QuickSortStrategy());
    } else {
      // Use merge sort for large arrays
      this.setStrategy(new MergeSortStrategy());
    }

    return this.strategy.sort(data);
  }
}

// Usage
const sorter = new DataSorter(new QuickSortStrategy());
const data = [64, 34, 25, 12, 22, 11, 90];
const sorted = sorter.sort(data);
console.log(sorted);
```

### Compression Strategies

```javascript
class CompressionStrategy {
  compress(data) {
    throw new Error('compress() must be implemented');
  }

  decompress(data) {
    throw new Error('decompress() must be implemented');
  }
}

class ZipCompression extends CompressionStrategy {
  compress(data) {
    console.log('Compressing using ZIP algorithm');
    // Simulate ZIP compression
    return {
      algorithm: 'ZIP',
      data: `[ZIP_COMPRESSED: ${data.substring(0, 10)}...]`,
      originalSize: data.length,
      compressedSize: Math.floor(data.length * 0.6)
    };
  }

  decompress(compressed) {
    console.log('Decompressing ZIP data');
    return compressed.data;
  }
}

class RARCompression extends CompressionStrategy {
  compress(data) {
    console.log('Compressing using RAR algorithm');
    return {
      algorithm: 'RAR',
      data: `[RAR_COMPRESSED: ${data.substring(0, 10)}...]`,
      originalSize: data.length,
      compressedSize: Math.floor(data.length * 0.5)
    };
  }

  decompress(compressed) {
    console.log('Decompressing RAR data');
    return compressed.data;
  }
}

class GzipCompression extends CompressionStrategy {
  compress(data) {
    console.log('Compressing using GZIP algorithm');
    return {
      algorithm: 'GZIP',
      data: `[GZIP_COMPRESSED: ${data.substring(0, 10)}...]`,
      originalSize: data.length,
      compressedSize: Math.floor(data.length * 0.7)
    };
  }

  decompress(compressed) {
    console.log('Decompressing GZIP data');
    return compressed.data;
  }
}

// Context
class FileCompressor {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  compressFile(data) {
    const result = this.strategy.compress(data);
    const ratio = ((1 - result.compressedSize / result.originalSize) * 100).toFixed(2);
    console.log(`Compression ratio: ${ratio}%`);
    return result;
  }

  decompressFile(compressed) {
    return this.strategy.decompress(compressed);
  }
}

// Usage
const data = 'This is some data that needs to be compressed. '.repeat(10);

const compressor = new FileCompressor(new ZipCompression());
let compressed = compressor.compressFile(data);

// Switch strategy
compressor.setStrategy(new RARCompression());
compressed = compressor.compressFile(data);

// Choose based on file type
function selectCompression(fileType) {
  switch (fileType) {
    case '.zip': return new ZipCompression();
    case '.rar': return new RARCompression();
    case '.gz': return new GzipCompression();
    default: return new ZipCompression();
  }
}

const strategy = selectCompression('.rar');
compressor.setStrategy(strategy);
```

### Validation Strategies

```javascript
class ValidationStrategy {
  validate(value) {
    throw new Error('validate() must be implemented');
  }
}

class EmailValidation extends ValidationStrategy {
  validate(email) {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const isValid = regex.test(email);
    return {
      isValid,
      message: isValid ? 'Valid email' : 'Invalid email format'
    };
  }
}

class PhoneValidation extends ValidationStrategy {
  validate(phone) {
    const regex = /^\+?1?\d{10,14}$/;
    const isValid = regex.test(phone.replace(/[\s-()]/g, ''));
    return {
      isValid,
      message: isValid ? 'Valid phone' : 'Invalid phone format'
    };
  }
}

class PasswordValidation extends ValidationStrategy {
  constructor(minLength = 8) {
    super();
    this.minLength = minLength;
  }

  validate(password) {
    const checks = {
      length: password.length >= this.minLength,
      uppercase: /[A-Z]/.test(password),
      lowercase: /[a-z]/.test(password),
      number: /\d/.test(password),
      special: /[!@#$%^&*]/.test(password)
    };

    const isValid = Object.values(checks).every(check => check);

    const messages = [];
    if (!checks.length) messages.push(`At least ${this.minLength} characters`);
    if (!checks.uppercase) messages.push('One uppercase letter');
    if (!checks.lowercase) messages.push('One lowercase letter');
    if (!checks.number) messages.push('One number');
    if (!checks.special) messages.push('One special character');

    return {
      isValid,
      message: isValid ? 'Valid password' : `Password must contain: ${messages.join(', ')}`
    };
  }
}

// Context
class FormValidator {
  constructor() {
    this.strategies = new Map();
  }

  setStrategy(fieldName, strategy) {
    this.strategies.set(fieldName, strategy);
  }

  validate(fieldName, value) {
    const strategy = this.strategies.get(fieldName);
    if (!strategy) {
      throw new Error(`No validation strategy for field: ${fieldName}`);
    }
    return strategy.validate(value);
  }

  validateForm(formData) {
    const results = {};
    const errors = [];

    for (const [field, value] of Object.entries(formData)) {
      const result = this.validate(field, value);
      results[field] = result;

      if (!result.isValid) {
        errors.push(`${field}: ${result.message}`);
      }
    }

    return {
      isValid: errors.length === 0,
      results,
      errors
    };
  }
}

// Usage
const validator = new FormValidator();
validator.setStrategy('email', new EmailValidation());
validator.setStrategy('phone', new PhoneValidation());
validator.setStrategy('password', new PasswordValidation(10));

const formData = {
  email: 'user@example.com',
  phone: '+1234567890',
  password: 'Weak123'
};

const result = validator.validateForm(formData);
console.log('Form valid:', result.isValid);
result.errors.forEach(error => console.log('Error:', error));
```

### Pricing Strategies

```javascript
class PricingStrategy {
  calculatePrice(order) {
    throw new Error('calculatePrice() must be implemented');
  }
}

class RegularPricing extends PricingStrategy {
  calculatePrice(order) {
    const subtotal = order.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    return {
      subtotal,
      discount: 0,
      total: subtotal,
      strategy: 'Regular'
    };
  }
}

class BlackFridayPricing extends PricingStrategy {
  calculatePrice(order) {
    const subtotal = order.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    const discount = subtotal * 0.50; // 50% off
    return {
      subtotal,
      discount,
      total: subtotal - discount,
      strategy: 'Black Friday'
    };
  }
}

class LoyaltyPricing extends PricingStrategy {
  constructor(loyaltyPoints) {
    super();
    this.loyaltyPoints = loyaltyPoints;
  }

  calculatePrice(order) {
    const subtotal = order.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    const discountPercent = Math.min(this.loyaltyPoints / 100, 0.30); // Max 30%
    const discount = subtotal * discountPercent;
    return {
      subtotal,
      discount,
      total: subtotal - discount,
      strategy: 'Loyalty',
      pointsUsed: this.loyaltyPoints
    };
  }
}

class BulkPricing extends PricingStrategy {
  calculatePrice(order) {
    const totalItems = order.items.reduce((sum, item) => sum + item.quantity, 0);
    const subtotal = order.items.reduce((sum, item) => sum + item.price * item.quantity, 0);

    let discountPercent = 0;
    if (totalItems >= 100) discountPercent = 0.20;
    else if (totalItems >= 50) discountPercent = 0.15;
    else if (totalItems >= 20) discountPercent = 0.10;

    const discount = subtotal * discountPercent;
    return {
      subtotal,
      discount,
      total: subtotal - discount,
      strategy: 'Bulk',
      itemCount: totalItems
    };
  }
}

// Context
class Order {
  constructor() {
    this.items = [];
    this.pricingStrategy = new RegularPricing();
  }

  addItem(item) {
    this.items.push(item);
  }

  setPricingStrategy(strategy) {
    this.pricingStrategy = strategy;
  }

  calculateTotal() {
    return this.pricingStrategy.calculatePrice(this);
  }
}

// Usage
const order = new Order();
order.addItem({ name: 'Laptop', price: 1000, quantity: 1 });
order.addItem({ name: 'Mouse', price: 50, quantity: 2 });

// Regular pricing
console.log(order.calculateTotal());

// Black Friday
order.setPricingStrategy(new BlackFridayPricing());
console.log(order.calculateTotal());

// Loyalty member
order.setPricingStrategy(new LoyaltyPricing(5000));
console.log(order.calculateTotal());

// Bulk order
const bulkOrder = new Order();
bulkOrder.addItem({ name: 'Widget', price: 10, quantity: 60 });
bulkOrder.setPricingStrategy(new BulkPricing());
console.log(bulkOrder.calculateTotal());
```

## Benefits

- **Open/Closed Principle**: Add new strategies without modifying context
- **Runtime Flexibility**: Switch algorithms at runtime
- **Eliminates Conditionals**: Replaces complex if/else or switch statements
- **Encapsulation**: Algorithm details hidden from client
- **Testability**: Each strategy can be tested independently
- **Reusability**: Strategies can be reused across different contexts

## Drawbacks

- **Increased Objects**: More classes/objects to manage
- **Client Awareness**: Client must understand different strategies
- **Communication Overhead**: Might need to pass unused data to strategies
- **Overkill**: Can be overkill for simple variations

## When to Use

✅ **Good Use Cases**:
- Multiple algorithms for same task
- Need to switch algorithms at runtime
- Many conditional statements based on type
- Want to hide algorithm complexity
- Algorithm varies based on context

❌ **Avoid When**:
- Only one or two variations
- Algorithms rarely change
- Simple conditional logic
- Performance is critical (extra indirection)

## Best Practices

1. **Common Interface**: All strategies should share common interface
2. **Context Independence**: Strategies shouldn't depend on context details
3. **Factory Pattern**: Use factory to create strategies
4. **Default Strategy**: Provide sensible default
5. **Strategy Selection**: Encapsulate selection logic
6. **Null Strategy**: Consider null object pattern for optional strategies

### With Factory Pattern

```javascript
class StrategyFactory {
  static createPaymentStrategy(type, details) {
    switch (type) {
      case 'credit_card':
        return new CreditCardStrategy(details.number, details.cvv);
      case 'paypal':
        return new PayPalStrategy(details.email);
      case 'bitcoin':
        return new BitcoinStrategy(details.wallet);
      default:
        throw new Error(`Unknown payment type: ${type}`);
    }
  }
}

// Usage
const strategy = StrategyFactory.createPaymentStrategy('paypal', {
  email: 'user@example.com'
});
cart.setPaymentStrategy(strategy);
```

## Related Patterns

- **State**: Similar structure, but State changes behavior based on state
- **Command**: Encapsulates requests as objects
- **Template Method**: Defines algorithm skeleton, subclasses override steps
- **Factory**: Can create strategies

## Comparison with State Pattern

| Aspect | Strategy | State |
|--------|----------|-------|
| Purpose | Choose algorithm | Change behavior based on state |
| Context Awareness | Strategy independent | State knows context |
| Transitions | Client switches | State can switch itself |
| Focus | Interchangeable algorithms | Object lifecycle |

## Common Mistakes

```javascript
// ❌ Bad: Tight coupling
class BadContext {
  process() {
    if (this.type === 'A') {
      // Algorithm A
    } else if (this.type === 'B') {
      // Algorithm B
    }
    // Hard to extend
  }
}

// ✅ Good: Strategy pattern
class GoodContext {
  constructor(strategy) {
    this.strategy = strategy;
  }

  process() {
    return this.strategy.execute();
  }
}

// ❌ Bad: Strategies know too much about context
class BadStrategy {
  execute(context) {
    context.data.forEach(...); // Knows context internals
  }
}

// ✅ Good: Strategy receives only what it needs
class GoodStrategy {
  execute(data) {
    data.forEach(...); // Only knows about data
  }
}
```
