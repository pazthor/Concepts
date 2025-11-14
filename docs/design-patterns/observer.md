# Observer Pattern

## Category
Behavioral Pattern

## Overview

The Observer pattern defines a one-to-many dependency between objects where when one object (subject) changes state, all its dependents (observers) are automatically notified and updated. Also known as Publish-Subscribe pattern.

## Problem

- Need to notify multiple objects about state changes
- Want loose coupling between objects
- Number of observers can vary dynamically
- Don't want tight dependencies between subject and observers

## Solution

Create a subscription mechanism where:
1. Subject maintains list of observers
2. Observers subscribe/unsubscribe to subject
3. Subject notifies all observers when state changes
4. Observers update themselves based on notification

## Structure

```
Subject
├── attach(observer)
├── detach(observer)
├── notify()
└── observers[]

Observer (Interface)
└── update()

ConcreteObserverA    ConcreteObserverB
└── update()         └── update()
```

## Implementation

### Basic Observer

```javascript
// Subject
class Subject {
  constructor() {
    this.observers = [];
  }

  attach(observer) {
    if (!this.observers.includes(observer)) {
      this.observers.push(observer);
    }
  }

  detach(observer) {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }

  notify(data) {
    this.observers.forEach(observer => {
      observer.update(data);
    });
  }
}

// Observer interface
class Observer {
  update(data) {
    throw new Error('update() must be implemented');
  }
}

// Concrete observers
class ConcreteObserverA extends Observer {
  update(data) {
    console.log(`Observer A received: ${data}`);
  }
}

class ConcreteObserverB extends Observer {
  update(data) {
    console.log(`Observer B received: ${data}`);
  }
}

// Usage
const subject = new Subject();
const observerA = new ConcreteObserverA();
const observerB = new ConcreteObserverB();

subject.attach(observerA);
subject.attach(observerB);

subject.notify('Hello Observers!');
// Observer A received: Hello Observers!
// Observer B received: Hello Observers!

subject.detach(observerA);
subject.notify('Another message');
// Observer B received: Another message
```

### Event Emitter Pattern

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
    return this;
  }

  off(event, callback) {
    if (!this.events[event]) return this;

    if (!callback) {
      delete this.events[event];
    } else {
      this.events[event] = this.events[event].filter(cb => cb !== callback);
    }
    return this;
  }

  emit(event, ...args) {
    if (!this.events[event]) return this;

    this.events[event].forEach(callback => {
      callback(...args);
    });
    return this;
  }

  once(event, callback) {
    const onceCallback = (...args) => {
      callback(...args);
      this.off(event, onceCallback);
    };
    return this.on(event, onceCallback);
  }
}

// Usage
const emitter = new EventEmitter();

const handler1 = data => console.log(`Handler 1: ${data}`);
const handler2 = data => console.log(`Handler 2: ${data}`);

emitter.on('message', handler1);
emitter.on('message', handler2);

emitter.emit('message', 'Hello!');
// Handler 1: Hello!
// Handler 2: Hello!

emitter.off('message', handler1);
emitter.emit('message', 'Again!');
// Handler 2: Again!

emitter.once('oneTime', () => console.log('Once only'));
emitter.emit('oneTime'); // Once only
emitter.emit('oneTime'); // Nothing happens
```

## Real-World Examples

### News Agency (Publisher-Subscriber)

```javascript
class NewsAgency {
  constructor() {
    this.subscribers = [];
    this.latestNews = null;
  }

  subscribe(subscriber) {
    this.subscribers.push(subscriber);
    console.log(`${subscriber.name} subscribed`);
  }

  unsubscribe(subscriber) {
    this.subscribers = this.subscribers.filter(sub => sub !== subscriber);
    console.log(`${subscriber.name} unsubscribed`);
  }

  publishNews(news) {
    console.log(`\nPublishing: ${news.headline}`);
    this.latestNews = news;
    this.notifySubscribers(news);
  }

  notifySubscribers(news) {
    this.subscribers.forEach(subscriber => {
      subscriber.update(news);
    });
  }
}

class NewsSubscriber {
  constructor(name) {
    this.name = name;
  }

  update(news) {
    console.log(`${this.name} received: ${news.headline}`);
    this.displayNews(news);
  }

  displayNews(news) {
    console.log(`  [${this.name}] ${news.content}`);
  }
}

// Usage
const agency = new NewsAgency();

const subscriber1 = new NewsSubscriber('Mobile App');
const subscriber2 = new NewsSubscriber('Website');
const subscriber3 = new NewsSubscriber('Email Service');

agency.subscribe(subscriber1);
agency.subscribe(subscriber2);
agency.subscribe(subscriber3);

agency.publishNews({
  headline: 'Breaking News!',
  content: 'Something important happened'
});

agency.unsubscribe(subscriber2);

agency.publishNews({
  headline: 'Update',
  content: 'Another story'
});
```

### Stock Market Tracker

```javascript
class Stock {
  constructor(symbol, price) {
    this.symbol = symbol;
    this.price = price;
    this.observers = [];
  }

  attach(observer) {
    this.observers.push(observer);
  }

  detach(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
  }

  setPrice(newPrice) {
    const oldPrice = this.price;
    this.price = newPrice;
    const change = ((newPrice - oldPrice) / oldPrice * 100).toFixed(2);

    this.notify({
      symbol: this.symbol,
      oldPrice,
      newPrice,
      change: `${change}%`
    });
  }

  notify(data) {
    this.observers.forEach(observer => {
      observer.update(data);
    });
  }
}

class StockDisplay {
  constructor(name) {
    this.name = name;
  }

  update(data) {
    console.log(`\n[${this.name}]`);
    console.log(`${data.symbol}: $${data.oldPrice} → $${data.newPrice}`);
    console.log(`Change: ${data.change}`);
  }
}

class StockAlert {
  constructor(threshold) {
    this.threshold = threshold;
  }

  update(data) {
    const changePercent = parseFloat(data.change);
    if (Math.abs(changePercent) >= this.threshold) {
      console.log(`\n🚨 ALERT: ${data.symbol} changed by ${data.change}!`);
    }
  }
}

class StockLogger {
  constructor() {
    this.history = [];
  }

  update(data) {
    this.history.push({
      timestamp: new Date(),
      ...data
    });
  }

  printHistory() {
    console.log('\n=== Trade History ===');
    this.history.forEach(entry => {
      console.log(`${entry.timestamp.toLocaleTimeString()}: ${entry.symbol} @ $${entry.newPrice}`);
    });
  }
}

// Usage
const appleStock = new Stock('AAPL', 150);

const mobileDisplay = new StockDisplay('Mobile App');
const webDisplay = new StockDisplay('Website');
const alert = new StockAlert(5); // Alert on 5% change
const logger = new StockLogger();

appleStock.attach(mobileDisplay);
appleStock.attach(webDisplay);
appleStock.attach(alert);
appleStock.attach(logger);

appleStock.setPrice(155);  // +3.33%
appleStock.setPrice(165);  // +6.45% - triggers alert
appleStock.setPrice(160);  // -3.03%

logger.printHistory();
```

### UI Component State Management

```javascript
class StateManager extends EventEmitter {
  constructor(initialState = {}) {
    super();
    this.state = initialState;
    this.previousState = null;
  }

  getState() {
    return { ...this.state };
  }

  setState(updates) {
    this.previousState = { ...this.state };
    this.state = { ...this.state, ...updates };

    this.emit('stateChange', {
      current: this.state,
      previous: this.previousState,
      changes: updates
    });
  }

  subscribe(callback) {
    return this.on('stateChange', callback);
  }
}

// UI Components as observers
class ComponentA {
  constructor(stateManager) {
    this.stateManager = stateManager;
    this.stateManager.subscribe(this.handleStateChange.bind(this));
  }

  handleStateChange({ current, changes }) {
    if ('count' in changes) {
      this.render(current.count);
    }
  }

  render(count) {
    console.log(`ComponentA: Count is ${count}`);
  }
}

class ComponentB {
  constructor(stateManager) {
    this.stateManager = stateManager;
    this.stateManager.subscribe(this.handleStateChange.bind(this));
  }

  handleStateChange({ current, changes }) {
    if ('user' in changes) {
      this.render(current.user);
    }
  }

  render(user) {
    console.log(`ComponentB: User is ${user}`);
  }
}

// Usage
const store = new StateManager({ count: 0, user: 'Guest' });

const compA = new ComponentA(store);
const compB = new ComponentB(store);

store.setState({ count: 1 });    // ComponentA updates
store.setState({ user: 'Alice' }); // ComponentB updates
store.setState({ count: 2, user: 'Bob' }); // Both update
```

### Chat Room

```javascript
class ChatRoom {
  constructor(name) {
    this.name = name;
    this.users = new Set();
  }

  join(user) {
    this.users.add(user);
    user.chatRoom = this;
    this.broadcast(`${user.name} joined the room`, user);
  }

  leave(user) {
    this.users.delete(user);
    user.chatRoom = null;
    this.broadcast(`${user.name} left the room`, user);
  }

  broadcast(message, sender) {
    this.users.forEach(user => {
      if (user !== sender) {
        user.receive(message, sender);
      }
    });
  }

  sendMessage(message, sender) {
    console.log(`\n[${this.name}] ${sender.name}: ${message}`);
    this.broadcast(message, sender);
  }
}

class User {
  constructor(name) {
    this.name = name;
    this.chatRoom = null;
  }

  send(message) {
    if (!this.chatRoom) {
      console.log(`${this.name} is not in a chat room`);
      return;
    }
    this.chatRoom.sendMessage(message, this);
  }

  receive(message, sender) {
    console.log(`  ${this.name} received from ${sender?.name || 'System'}: ${message}`);
  }
}

// Usage
const chatRoom = new ChatRoom('General');

const alice = new User('Alice');
const bob = new User('Bob');
const charlie = new User('Charlie');

chatRoom.join(alice);
chatRoom.join(bob);
chatRoom.join(charlie);

alice.send('Hello everyone!');
bob.send('Hi Alice!');

chatRoom.leave(charlie);
alice.send('Charlie left');
```

## Benefits

- **Loose Coupling**: Subject and observers are loosely coupled
- **Dynamic Relationships**: Observers can be added/removed at runtime
- **Broadcast Communication**: One-to-many communication
- **Open/Closed Principle**: Add new observers without modifying subject
- **Reusability**: Observers can be reused with different subjects

## Drawbacks

- **Memory Leaks**: Forgotten subscriptions can cause leaks
- **Unexpected Updates**: Hard to track update chains
- **Performance**: Many observers = many updates
- **Order Dependency**: Notification order might matter
- **Debugging**: Hard to debug cascade effects

## When to Use

✅ **Good Use Cases**:
- State changes need to notify multiple objects
- Number of observers varies dynamically
- Event handling systems
- MVC architecture (Model notifies Views)
- Real-time notifications
- UI updates based on data changes

❌ **Avoid When**:
- Simple one-to-one relationships
- Performance is critical and many observers exist
- Order of execution is important

## Best Practices

1. **Cleanup**: Always unsubscribe when done
2. **Weak References**: Consider weak references to prevent leaks
3. **Error Handling**: Don't let one observer crash others
4. **Async Notifications**: Consider async for heavy operations
5. **Event Types**: Use specific event types
6. **Documentation**: Document notification contracts

### Safe Observer Implementation

```javascript
class SafeSubject {
  constructor() {
    this.observers = [];
  }

  attach(observer) {
    this.observers.push(observer);
    // Return unsubscribe function
    return () => this.detach(observer);
  }

  detach(observer) {
    this.observers = this.observers.filter(obs => obs !== observer);
  }

  async notify(data) {
    // Create copy to avoid modification during iteration
    const observers = [...this.observers];

    for (const observer of observers) {
      try {
        // Catch errors to prevent one observer from crashing others
        await observer.update(data);
      } catch (error) {
        console.error('Observer error:', error);
      }
    }
  }
}

// Usage with auto-cleanup
const subject = new SafeSubject();
const unsubscribe = subject.attach(observer);

// Later
unsubscribe(); // Clean up
```

## Related Patterns

- **Mediator**: Centralizes complex communications
- **Singleton**: Subject can be a singleton
- **Event Sourcing**: Stores events for replay
- **Command**: Can be used with observer for undo/redo

## Common Mistakes

```javascript
// ❌ Bad: Memory leak - no cleanup
class Component {
  constructor(subject) {
    subject.attach(this);
    // Never detaches!
  }
}

// ✅ Good: Proper cleanup
class Component {
  constructor(subject) {
    this.subject = subject;
    this.subject.attach(this);
  }

  destroy() {
    this.subject.detach(this);
  }
}

// ❌ Bad: Order matters
observers.forEach(o => o.update());
// If observers depend on each other, problems occur

// ✅ Good: Use priority or dependency management
class PrioritySubject {
  notify() {
    const sorted = this.observers.sort((a, b) => a.priority - b.priority);
    sorted.forEach(o => o.update());
  }
}
```
