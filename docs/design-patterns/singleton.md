# Singleton Pattern

## Category
Creational Pattern

## Overview

The Singleton pattern ensures that a class has only one instance and provides a global point of access to that instance. It's useful when exactly one object is needed to coordinate actions across the system.

## Problem

Sometimes you need exactly one instance of a class:
- Database connections
- Configuration managers
- Logger instances
- Thread pools
- Cache managers

Multiple instances could cause problems like:
- Conflicting state
- Resource wastage
- Inconsistent behavior

## Solution

Make the class responsible for keeping track of its sole instance:
1. Make constructor private to prevent direct instantiation
2. Create static method to get the instance
3. Lazy or eager initialization

## Implementation

### Basic Singleton

```javascript
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }
    Singleton.instance = this;
    // Initialize your singleton
    this.data = {};
  }

  static getInstance() {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton();
    }
    return Singleton.instance;
  }

  setData(key, value) {
    this.data[key] = value;
  }

  getData(key) {
    return this.data[key];
  }
}

// Usage
const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();

console.log(instance1 === instance2); // true

instance1.setData('user', 'Alice');
console.log(instance2.getData('user')); // 'Alice'
```

### Modern ES6 Module Singleton

```javascript
// config.js
class Config {
  constructor() {
    this.settings = {
      apiUrl: 'https://api.example.com',
      timeout: 5000
    };
  }

  get(key) {
    return this.settings[key];
  }

  set(key, value) {
    this.settings[key] = value;
  }
}

// Export single instance
export default new Config();

// Usage in other files
import config from './config.js';

config.set('apiUrl', 'https://api.newurl.com');
console.log(config.get('apiUrl'));
```

### Thread-Safe Singleton (Conceptual)

```javascript
class ThreadSafeSingleton {
  constructor() {
    if (ThreadSafeSingleton.instance) {
      return ThreadSafeSingleton.instance;
    }

    if (ThreadSafeSingleton.isCreating) {
      throw new Error('Singleton is already being created');
    }

    ThreadSafeSingleton.isCreating = true;

    // Initialize
    this.data = {};

    ThreadSafeSingleton.instance = this;
    ThreadSafeSingleton.isCreating = false;
  }

  static getInstance() {
    if (!ThreadSafeSingleton.instance) {
      new ThreadSafeSingleton();
    }
    return ThreadSafeSingleton.instance;
  }
}
```

### Lazy Initialization

```javascript
class LazyS ingleton {
  constructor() {
    this.timestamp = Date.now();
  }

  static getInstance() {
    if (!LazyS ingleton.instance) {
      console.log('Creating new instance...');
      LazyS ingleton.instance = new LazyS ingleton();
    }
    return LazyS ingleton.instance;
  }
}

// Instance only created when first accessed
const instance = LazyS ingleton.getInstance(); // "Creating new instance..."
const instance2 = LazyS ingleton.getInstance(); // No message
```

### Eager Initialization

```javascript
class EagerSingleton {
  constructor() {
    this.timestamp = Date.now();
  }

  static instance = new EagerSingleton(); // Created immediately

  static getInstance() {
    return EagerSingleton.instance;
  }
}

// Instance already exists
const instance = EagerSingleton.getInstance();
```

## Real-World Examples

### Database Connection

```javascript
class Database {
  constructor() {
    if (Database.instance) {
      return Database.instance;
    }

    this.connection = null;
    Database.instance = this;
  }

  connect(connectionString) {
    if (!this.connection) {
      console.log('Establishing database connection...');
      this.connection = {
        connectionString,
        connected: true,
        timestamp: Date.now()
      };
    }
    return this.connection;
  }

  query(sql) {
    if (!this.connection) {
      throw new Error('Not connected to database');
    }
    console.log(`Executing: ${sql}`);
    return { results: [] };
  }

  disconnect() {
    if (this.connection) {
      console.log('Closing database connection...');
      this.connection = null;
    }
  }

  static getInstance() {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }
}

// Usage
const db1 = Database.getInstance();
db1.connect('mongodb://localhost:27017/mydb');

const db2 = Database.getInstance();
db2.query('SELECT * FROM users'); // Uses same connection

console.log(db1 === db2); // true
```

### Logger

```javascript
class Logger {
  constructor() {
    if (Logger.instance) {
      return Logger.instance;
    }

    this.logs = [];
    Logger.instance = this;
  }

  log(message, level = 'INFO') {
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message
    };
    this.logs.push(entry);
    console.log(`[${entry.timestamp}] ${level}: ${message}`);
  }

  error(message) {
    this.log(message, 'ERROR');
  }

  warn(message) {
    this.log(message, 'WARN');
  }

  info(message) {
    this.log(message, 'INFO');
  }

  getLogs() {
    return this.logs;
  }

  clearLogs() {
    this.logs = [];
  }

  static getInstance() {
    if (!Logger.instance) {
      Logger.instance = new Logger();
    }
    return Logger.instance;
  }
}

// Usage across application
const logger = Logger.getInstance();
logger.info('Application started');
logger.error('Something went wrong');

// In another module
const sameLogger = Logger.getInstance();
console.log(sameLogger.getLogs().length); // Shows all logs
```

### Configuration Manager

```javascript
class ConfigManager {
  constructor() {
    if (ConfigManager.instance) {
      return ConfigManager.instance;
    }

    this.config = new Map();
    this.loadDefaultConfig();
    ConfigManager.instance = this;
  }

  loadDefaultConfig() {
    this.config.set('env', 'development');
    this.config.set('debug', true);
    this.config.set('apiUrl', 'http://localhost:3000');
    this.config.set('maxRetries', 3);
  }

  get(key) {
    return this.config.get(key);
  }

  set(key, value) {
    this.config.set(key, value);
  }

  has(key) {
    return this.config.has(key);
  }

  getAll() {
    return Object.fromEntries(this.config);
  }

  loadFromFile(configData) {
    Object.entries(configData).forEach(([key, value]) => {
      this.config.set(key, value);
    });
  }

  static getInstance() {
    if (!ConfigManager.instance) {
      ConfigManager.instance = new ConfigManager();
    }
    return ConfigManager.instance;
  }
}

// Usage
const config = ConfigManager.getInstance();
console.log(config.get('apiUrl'));

// Later in another module
const sameConfig = ConfigManager.getInstance();
sameConfig.set('apiUrl', 'https://api.production.com');
```

## Benefits

- **Controlled Access**: Single point of access to the instance
- **Reduced Global Variables**: Better than global variables
- **Lazy Initialization**: Create only when needed
- **Consistency**: Single source of truth

## Drawbacks

- **Global State**: Can make testing difficult
- **Hidden Dependencies**: Not clear from function signatures
- **Tight Coupling**: Code becomes coupled to singleton
- **Difficult to Test**: Hard to mock or replace
- **Violates Single Responsibility**: Class manages both its logic and instance creation
- **Thread Safety**: Can be problematic in multi-threaded environments
- **Anti-pattern**: Often considered an anti-pattern in modern development

## When to Use

✅ **Good Use Cases**:
- Logger instances
- Configuration managers
- Database connections
- Thread pools
- Cache managers
- Device drivers

❌ **Bad Use Cases**:
- When you need multiple instances later
- For domain objects
- When it makes testing harder
- When dependency injection would be better

## Alternatives

### Dependency Injection

```javascript
// Instead of Singleton
class UserService {
  constructor() {
    this.db = Database.getInstance(); // Tight coupling
  }
}

// Better: Dependency Injection
class UserService {
  constructor(database) {
    this.db = database; // Loose coupling, testable
  }
}

// Usage
const db = new Database();
const userService = new UserService(db);
```

### Module Pattern (JavaScript)

```javascript
// module.js
let instance = null;

export function getLogger() {
  if (!instance) {
    instance = {
      logs: [],
      log(message) {
        this.logs.push(message);
        console.log(message);
      }
    };
  }
  return instance;
}
```

## Testing Considerations

```javascript
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }
    this.data = {};
    Singleton.instance = this;
  }

  // Add reset for testing
  static reset() {
    Singleton.instance = null;
  }

  static getInstance() {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton();
    }
    return Singleton.instance;
  }
}

// In tests
afterEach(() => {
  Singleton.reset(); // Clean state between tests
});

test('singleton behavior', () => {
  const instance1 = Singleton.getInstance();
  const instance2 = Singleton.getInstance();
  expect(instance1).toBe(instance2);
});
```

## Related Patterns

- **Factory Pattern**: Can use Singleton to ensure single factory
- **Abstract Factory**: Factory can be a Singleton
- **Facade**: Facade can be implemented as Singleton
- **State**: State objects can be Singletons

## Best Practices

1. **Make Constructor Private**: Prevent direct instantiation
2. **Lazy Initialization**: Create only when needed
3. **Thread Safety**: Consider concurrent access
4. **Testing**: Provide reset mechanism for tests
5. **Consider Alternatives**: DI might be better
6. **Document**: Clearly document why Singleton is needed
7. **Avoid Overuse**: Don't use as default pattern

## Common Mistakes

```javascript
// ❌ Bad: Can create multiple instances
class BadSingleton {
  static getInstance() {
    return new BadSingleton(); // Always creates new!
  }
}

// ❌ Bad: Subclassing issues
class ExtendedSingleton extends Singleton {
  // Creates separate instances
}

// ❌ Bad: Not actually restricting instantiation
class FakeSingleton {
  static instance = null;

  static getInstance() {
    if (!this.instance) {
      this.instance = new FakeSingleton();
    }
    return this.instance;
  }
}

// Can still do: new FakeSingleton() - not restricted!

// ✅ Good: Properly restricted
class RealSingleton {
  constructor() {
    if (RealSingleton.instance) {
      throw new Error('Use getInstance() instead');
    }
    RealSingleton.instance = this;
  }

  static getInstance() {
    if (!RealSingleton.instance) {
      new RealSingleton();
    }
    return RealSingleton.instance;
  }
}
```
