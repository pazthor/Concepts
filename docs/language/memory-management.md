# Memory Management

## Overview

Memory management is the process of allocating, using, and freeing memory in a program. Different languages handle memory management differently, from manual to automatic.

## Memory Allocation Strategies

### Stack Allocation

Fast, automatic allocation for local variables.

```javascript
function example() {
  const x = 10;        // Allocated on stack
  const y = 20;        // Allocated on stack
  const sum = x + y;   // Allocated on stack
} // All variables freed when function exits

// Stack memory characteristics:
// - LIFO (Last In, First Out)
// - Fast allocation/deallocation
// - Limited size
// - Automatic cleanup
```

### Heap Allocation

Dynamic allocation for objects and data structures.

```javascript
function createUser() {
  const user = {      // Object allocated on heap
    name: 'Alice',
    age: 30,
    profile: {        // Nested object also on heap
      bio: 'Software developer'
    }
  };
  return user;        // Reference to heap object returned
}

const user = createUser(); // Heap object still accessible

// Heap memory characteristics:
// - Dynamic allocation
// - Slower than stack
// - Larger size
// - Manual or automatic cleanup (GC)
```

## Garbage Collection

Automatic memory management that reclaims unused memory.

### Reference Counting

Track number of references to each object.

```javascript
let obj1 = { data: 'A' };   // ref count = 1
let obj2 = obj1;             // ref count = 2
obj1 = null;                 // ref count = 1
obj2 = null;                 // ref count = 0, can be collected

// Problem: Circular references
let a = { data: 'A' };
let b = { data: 'B' };
a.ref = b;                   // a references b
b.ref = a;                   // b references a (cycle!)
a = null;
b = null;
// Memory leak: Both have ref count > 0 despite being unreachable
```

### Mark and Sweep

JavaScript's primary GC algorithm.

```
1. Mark Phase:
   - Start from "roots" (global variables, call stack)
   - Recursively mark all reachable objects

2. Sweep Phase:
   - Scan memory for unmarked objects
   - Free memory of unmarked objects

┌────────────┐
│ Root       │
└──┬─────────┘
   │ (marks reachable)
   ▼
┌────────────┐     ┌────────────┐
│ Object A   │────▶│ Object B   │
│ (marked)   │     │ (marked)   │
└────────────┘     └────────────┘

┌────────────┐
│ Object C   │ (unreachable, swept away)
│ (unmarked) │
└────────────┘
```

```javascript
// Example of unreachable objects
function example() {
  let obj = { data: 'temporary' };
  // obj becomes unreachable after function returns
}

example();
// GC can now collect the object
```

### Generational GC

Optimize based on object lifetime.

```
Young Generation (frequent GC):
┌─────────────────────────────┐
│  New objects (short-lived)  │
└─────────────────────────────┘

Old Generation (infrequent GC):
┌─────────────────────────────┐
│ Long-lived objects          │
└─────────────────────────────┘

Hypothesis: Most objects die young
- Frequently collect young generation
- Rarely collect old generation
- Move survivors to old generation
```

## Memory Leaks

Unintended memory retention.

### Common Causes

#### 1. Forgotten Timers/Callbacks

```javascript
// Bad: Timer keeps reference
function setupTimer() {
  const largeData = new Array(1000000);

  setInterval(() => {
    console.log(largeData.length); // largeData never freed
  }, 1000);
}

// Good: Clear timer when done
function setupTimerFixed() {
  const largeData = new Array(1000000);

  const timerId = setInterval(() => {
    console.log(largeData.length);
  }, 1000);

  // Clear when component unmounts or is done
  return () => clearInterval(timerId);
}
```

#### 2. DOM References

```javascript
// Bad: Detached DOM nodes held in memory
let elements = [];

function addElement() {
  const el = document.createElement('div');
  document.body.appendChild(el);
  elements.push(el); // Reference kept
}

function removeElement() {
  const el = elements.pop();
  el.remove(); // Removed from DOM but still in array!
}

// Good: Clean up references
function removeElementFixed() {
  const el = elements.pop();
  el.remove();
  el = null; // Release reference
  elements = elements.filter(e => e !== el);
}
```

#### 3. Closures

```javascript
// Bad: Closure holds large data
function createClosure() {
  const largeData = new Array(1000000).fill('data');

  return function() {
    console.log('Hello'); // Entire largeData kept in closure
  };
}

const fn = createClosure(); // largeData never freed

// Good: Only keep what's needed
function createClosureFixed() {
  const largeData = new Array(1000000).fill('data');
  const needed = largeData[0];

  return function() {
    console.log(needed); // Only one value kept
  };
}
```

#### 4. Event Listeners

```javascript
// Bad: Listener not removed
class Component {
  constructor() {
    this.data = new Array(1000000);
    window.addEventListener('resize', this.handleResize.bind(this));
  }

  handleResize() {
    console.log(this.data.length);
  }

  // No cleanup - memory leak when component destroyed
}

// Good: Remove listeners
class ComponentFixed {
  constructor() {
    this.data = new Array(1000000);
    this.boundResize = this.handleResize.bind(this);
    window.addEventListener('resize', this.boundResize);
  }

  handleResize() {
    console.log(this.data.length);
  }

  destroy() {
    window.removeEventListener('resize', this.boundResize);
    this.data = null;
  }
}
```

#### 5. Global Variables

```javascript
// Bad: Globals never freed
window.cache = {};

function cacheData(key, data) {
  window.cache[key] = data; // Accumulates forever
}

// Good: Implement cleanup
class Cache {
  constructor(maxSize = 100) {
    this.cache = new Map();
    this.maxSize = maxSize;
  }

  set(key, value) {
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey); // Remove oldest
    }
    this.cache.set(key, value);
  }

  get(key) {
    return this.cache.get(key);
  }

  clear() {
    this.cache.clear();
  }
}
```

## Memory Optimization Techniques

### Object Pooling

Reuse objects instead of creating new ones.

```javascript
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.available = [];
    this.inUse = new Set();

    // Pre-allocate objects
    for (let i = 0; i < initialSize; i++) {
      this.available.push(createFn());
    }
  }

  acquire() {
    let obj = this.available.pop();

    if (!obj) {
      obj = this.createFn();
    }

    this.inUse.add(obj);
    return obj;
  }

  release(obj) {
    if (this.inUse.has(obj)) {
      this.inUse.delete(obj);
      this.resetFn(obj);
      this.available.push(obj);
    }
  }

  clear() {
    this.available = [];
    this.inUse.clear();
  }
}

// Usage
const particlePool = new ObjectPool(
  () => ({ x: 0, y: 0, vx: 0, vy: 0 }), // create
  obj => { obj.x = 0; obj.y = 0; }      // reset
);

function createParticle() {
  const particle = particlePool.acquire();
  particle.x = 100;
  particle.y = 100;
  return particle;
}

function destroyParticle(particle) {
  particlePool.release(particle);
}
```

### WeakMap and WeakSet

Weak references that don't prevent GC.

```javascript
// WeakMap: Keys can be garbage collected
const cache = new WeakMap();

function process(obj) {
  if (cache.has(obj)) {
    return cache.get(obj);
  }

  const result = expensiveComputation(obj);
  cache.set(obj, result); // Doesn't prevent obj from being GC'd
  return result;
}

let user = { name: 'Alice' };
process(user);
user = null; // Cache entry automatically removed

// WeakSet: Items can be garbage collected
const processedObjects = new WeakSet();

function processOnce(obj) {
  if (processedObjects.has(obj)) {
    return;
  }

  doProcessing(obj);
  processedObjects.add(obj);
}
```

### Limiting Data Size

```javascript
// Circular buffer for limited history
class CircularBuffer {
  constructor(maxSize) {
    this.buffer = new Array(maxSize);
    this.maxSize = maxSize;
    this.index = 0;
    this.size = 0;
  }

  add(item) {
    this.buffer[this.index] = item;
    this.index = (this.index + 1) % this.maxSize;
    this.size = Math.min(this.size + 1, this.maxSize);
  }

  getAll() {
    if (this.size < this.maxSize) {
      return this.buffer.slice(0, this.size);
    }
    return [
      ...this.buffer.slice(this.index),
      ...this.buffer.slice(0, this.index)
    ];
  }
}

// Usage: Limited event history
const eventHistory = new CircularBuffer(100);
eventHistory.add(event); // Old events automatically overwritten
```

### Lazy Loading

```javascript
class LazyData {
  constructor(loadFn) {
    this.loadFn = loadFn;
    this.data = null;
    this.loaded = false;
  }

  async get() {
    if (!this.loaded) {
      this.data = await this.loadFn();
      this.loaded = true;
    }
    return this.data;
  }

  clear() {
    this.data = null;
    this.loaded = false;
  }
}

// Usage
const largeDataset = new LazyData(() => fetchLargeDataset());

// Only loaded when needed
const data = await largeDataset.get();

// Free memory when done
largeDataset.clear();
```

## Monitoring Memory

### Chrome DevTools

```javascript
// Performance profiling
console.profile('MyOperation');
performOperation();
console.profileEnd('MyOperation');

// Memory snapshots
// 1. Take snapshot
// 2. Perform action
// 3. Take another snapshot
// 4. Compare to find leaks

// Memory usage
if (performance.memory) {
  console.log('Used JS Heap:', performance.memory.usedJSHeapSize);
  console.log('Total JS Heap:', performance.memory.totalJSHeapSize);
  console.log('Heap Limit:', performance.memory.jsHeapSizeLimit);
}
```

### Node.js Memory Monitoring

```javascript
// Check memory usage
const used = process.memoryUsage();
console.log({
  rss: `${Math.round(used.rss / 1024 / 1024)} MB`,           // Total memory
  heapTotal: `${Math.round(used.heapTotal / 1024 / 1024)} MB`, // Allocated heap
  heapUsed: `${Math.round(used.heapUsed / 1024 / 1024)} MB`,  // Used heap
  external: `${Math.round(used.external / 1024 / 1024)} MB`   // C++ objects
});

// Force garbage collection (with --expose-gc flag)
if (global.gc) {
  global.gc();
  console.log('GC executed');
}

// Increase memory limit
// node --max-old-space-size=4096 app.js
```

## Best Practices

1. **Clean Up Resources**: Remove listeners, clear timers, close connections
2. **Avoid Global Variables**: Minimize global scope pollution
3. **Use WeakMap/WeakSet**: For caches with object keys
4. **Implement Object Pools**: For frequently created/destroyed objects
5. **Limit Collection Sizes**: Implement max sizes for caches
6. **Monitor Memory**: Profile regularly in development
7. **Handle Large Data**: Stream or chunk large datasets
8. **Clear References**: Set unused objects to null
9. **Use Lazy Loading**: Load data only when needed
10. **Test for Leaks**: Regular memory leak testing

## Memory Leak Detection

```javascript
// Simple leak detection
class LeakDetector {
  constructor() {
    this.baseline = null;
  }

  takeBaseline() {
    if (global.gc) global.gc();
    this.baseline = process.memoryUsage().heapUsed;
  }

  checkLeak(threshold = 10 * 1024 * 1024) { // 10 MB
    if (global.gc) global.gc();
    const current = process.memoryUsage().heapUsed;
    const diff = current - this.baseline;

    if (diff > threshold) {
      console.warn(`Possible leak: ${diff / 1024 / 1024} MB increase`);
      return true;
    }
    return false;
  }
}

// Usage
const detector = new LeakDetector();
detector.takeBaseline();

// Run operations
for (let i = 0; i < 1000; i++) {
  performOperation();
}

detector.checkLeak(); // Check if memory increased significantly
```

## Related Concepts

- Garbage Collection Algorithms
- Memory Fragmentation
- Reference Counting
- Smart Pointers (C++)
- RAII (Resource Acquisition Is Initialization)
- Memory Profiling
