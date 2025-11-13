# Concurrency and Parallelism

## Overview

**Concurrency**: Multiple tasks making progress by interleaving their execution (dealing with many things at once).

**Parallelism**: Multiple tasks executing simultaneously on different processors (doing many things at once).

```
Concurrency:     |--Task1--|  |--Task1--|
                   |--Task2--|  |--Task2--|

Parallelism:     |--Task1--|
                 |--Task2--|  (same time)
```

## Key Concepts

### Processes vs Threads

#### Processes
- Independent execution units
- Separate memory space
- Heavy resource usage
- Inter-process communication (IPC) needed
- More isolation and security

#### Threads
- Lightweight execution units within a process
- Share memory space
- Lower resource usage
- Direct memory access
- Less isolation

```javascript
// Node.js - Creating child processes
const { fork } = require('child_process');

const child = fork('worker.js');
child.send({ task: 'process data' });
child.on('message', result => {
  console.log('Result:', result);
});

// worker.js
process.on('message', task => {
  const result = processData(task);
  process.send(result);
});
```

### Asynchronous Programming

#### Callbacks

```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback(null, { data: 'result' });
  }, 1000);
}

fetchData((error, result) => {
  if (error) {
    console.error(error);
  } else {
    console.log(result);
  }
});
```

**Problems**: Callback hell, error handling complexity

#### Promises

```javascript
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({ data: 'result' });
    }, 1000);
  });
}

fetchData()
  .then(result => console.log(result))
  .catch(error => console.error(error));

// Chaining
fetchData()
  .then(result => processData(result))
  .then(processed => saveData(processed))
  .then(() => console.log('Done'))
  .catch(error => console.error(error));
```

#### Async/Await

```javascript
async function fetchAndProcess() {
  try {
    const data = await fetchData();
    const processed = await processData(data);
    await saveData(processed);
    console.log('Done');
  } catch (error) {
    console.error(error);
  }
}

// Parallel execution
async function fetchMultiple() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);
  return { user, posts, comments };
}
```

## Concurrency Patterns

### Event Loop

JavaScript's concurrency model.

```
┌───────────────────────────┐
│        Call Stack         │
└───────────┬───────────────┘
            │
┌───────────▼───────────────┐
│       Web APIs            │
│  (setTimeout, fetch, etc) │
└───────────┬───────────────┘
            │
┌───────────▼───────────────┐
│      Callback Queue       │
└───────────┬───────────────┘
            │
        Event Loop
            │
            └──────────▶ Back to Call Stack
```

```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
}, 0);

Promise.resolve().then(() => {
  console.log('3');
});

console.log('4');

// Output: 1, 4, 3, 2
// Microtasks (Promises) execute before macrotasks (setTimeout)
```

### Worker Threads (Node.js)

```javascript
// main.js
const { Worker } = require('worker_threads');

function runWorker(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js', {
      workerData: data
    });

    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', code => {
      if (code !== 0) {
        reject(new Error(`Worker stopped with exit code ${code}`));
      }
    });
  });
}

// Run multiple workers in parallel
async function processInParallel(items) {
  const workers = items.map(item => runWorker(item));
  const results = await Promise.all(workers);
  return results;
}

// worker.js
const { parentPort, workerData } = require('worker_threads');

// CPU-intensive task
const result = expensiveComputation(workerData);
parentPort.postMessage(result);
```

### Web Workers (Browser)

```javascript
// main.js
const worker = new Worker('worker.js');

worker.postMessage({ data: 'process this' });

worker.onmessage = event => {
  console.log('Result:', event.data);
};

worker.onerror = error => {
  console.error('Worker error:', error);
};

// worker.js
self.onmessage = event => {
  const result = processData(event.data);
  self.postMessage(result);
};
```

## Concurrency Challenges

### Race Conditions

Multiple operations on shared state without proper synchronization.

```javascript
// Bad: Race condition
let counter = 0;

async function increment() {
  const temp = counter;
  await delay(10); // Simulate async work
  counter = temp + 1;
}

// Both calls might read same value
await Promise.all([increment(), increment()]);
console.log(counter); // Might be 1 instead of 2!

// Good: Use atomic operations or locks
class Counter {
  constructor() {
    this.value = 0;
    this.lock = Promise.resolve();
  }

  async increment() {
    this.lock = this.lock.then(async () => {
      const temp = this.value;
      await delay(10);
      this.value = temp + 1;
    });
    await this.lock;
  }
}
```

### Deadlocks

Two or more operations waiting for each other.

```javascript
// Deadlock scenario
class Resource {
  constructor(name) {
    this.name = name;
    this.locked = false;
  }

  async lock() {
    while (this.locked) {
      await delay(10);
    }
    this.locked = true;
  }

  unlock() {
    this.locked = false;
  }
}

const resourceA = new Resource('A');
const resourceB = new Resource('B');

// Task 1: Lock A then B
async function task1() {
  await resourceA.lock();
  await delay(100);
  await resourceB.lock(); // Waiting for B
  resourceB.unlock();
  resourceA.unlock();
}

// Task 2: Lock B then A
async function task2() {
  await resourceB.lock();
  await delay(100);
  await resourceA.lock(); // Waiting for A - DEADLOCK!
  resourceA.unlock();
  resourceB.unlock();
}

// Solution: Always acquire locks in same order
async function task2Fixed() {
  await resourceA.lock(); // Same order as task1
  await resourceB.lock();
  resourceB.unlock();
  resourceA.unlock();
}
```

### Starvation

A task never gets resources it needs.

```javascript
// Priority queue to prevent starvation
class PriorityQueue {
  constructor() {
    this.tasks = [];
  }

  add(task, priority) {
    this.tasks.push({ task, priority, addedAt: Date.now() });
    this.tasks.sort((a, b) => {
      // Increase priority over time to prevent starvation
      const ageBonus = (Date.now() - a.addedAt) / 1000;
      return (b.priority + ageBonus) - a.priority;
    });
  }

  async process() {
    while (this.tasks.length > 0) {
      const { task } = this.tasks.shift();
      await task();
    }
  }
}
```

## Synchronization Primitives

### Mutex (Mutual Exclusion)

```javascript
class Mutex {
  constructor() {
    this.locked = false;
    this.queue = [];
  }

  async lock() {
    if (!this.locked) {
      this.locked = true;
      return;
    }

    return new Promise(resolve => {
      this.queue.push(resolve);
    });
  }

  unlock() {
    if (this.queue.length > 0) {
      const resolve = this.queue.shift();
      resolve();
    } else {
      this.locked = false;
    }
  }

  async runExclusive(callback) {
    await this.lock();
    try {
      return await callback();
    } finally {
      this.unlock();
    }
  }
}

// Usage
const mutex = new Mutex();

async function criticalSection() {
  await mutex.runExclusive(async () => {
    // Only one execution at a time
    await performOperation();
  });
}
```

### Semaphore

Controls access to limited resources.

```javascript
class Semaphore {
  constructor(max) {
    this.max = max;
    this.count = 0;
    this.queue = [];
  }

  async acquire() {
    if (this.count < this.max) {
      this.count++;
      return;
    }

    return new Promise(resolve => {
      this.queue.push(resolve);
    });
  }

  release() {
    if (this.queue.length > 0) {
      const resolve = this.queue.shift();
      resolve();
    } else {
      this.count--;
    }
  }

  async runWithLimit(callback) {
    await this.acquire();
    try {
      return await callback();
    } finally {
      this.release();
    }
  }
}

// Limit concurrent API calls
const apiSemaphore = new Semaphore(3); // Max 3 concurrent calls

async function makeApiCall(url) {
  return apiSemaphore.runWithLimit(async () => {
    const response = await fetch(url);
    return response.json();
  });
}

// Only 3 will run simultaneously
const results = await Promise.all(
  urls.map(url => makeApiCall(url))
);
```

### Channel Pattern

Communication between concurrent tasks.

```javascript
class Channel {
  constructor() {
    this.queue = [];
    this.receivers = [];
  }

  async send(value) {
    if (this.receivers.length > 0) {
      const receiver = this.receivers.shift();
      receiver(value);
    } else {
      return new Promise(resolve => {
        this.queue.push({ value, resolve });
      });
    }
  }

  async receive() {
    if (this.queue.length > 0) {
      const { value, resolve } = this.queue.shift();
      resolve();
      return value;
    }

    return new Promise(resolve => {
      this.receivers.push(resolve);
    });
  }
}

// Usage
const channel = new Channel();

// Producer
async function producer() {
  for (let i = 0; i < 10; i++) {
    await channel.send(i);
    console.log('Sent:', i);
  }
}

// Consumer
async function consumer() {
  while (true) {
    const value = await channel.receive();
    console.log('Received:', value);
  }
}

Promise.all([producer(), consumer()]);
```

## Async Patterns

### Throttling

Limit execution rate.

```javascript
function throttle(fn, delay) {
  let lastCall = 0;

  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      return fn(...args);
    }
  };
}

// Usage
const throttledScroll = throttle(() => {
  console.log('Scroll event processed');
}, 1000);

window.addEventListener('scroll', throttledScroll);
```

### Debouncing

Delay execution until after activity stops.

```javascript
function debounce(fn, delay) {
  let timeoutId;

  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      fn(...args);
    }, delay);
  };
}

// Usage
const debouncedSearch = debounce(query => {
  searchAPI(query);
}, 500);

input.addEventListener('input', e => {
  debouncedSearch(e.target.value);
});
```

### Retry Logic

```javascript
async function retry(fn, maxAttempts = 3, delay = 1000) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts) {
        throw error;
      }
      await new Promise(resolve => setTimeout(resolve, delay * attempt));
    }
  }
}

// Usage
const data = await retry(
  () => fetch('/api/data'),
  3,
  1000
);
```

### Circuit Breaker

Prevent cascading failures.

```javascript
class CircuitBreaker {
  constructor(threshold = 5, timeout = 60000) {
    this.failureCount = 0;
    this.threshold = threshold;
    this.timeout = timeout;
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.nextAttempt = Date.now();
  }

  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      this.state = 'HALF_OPEN';
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
    }
  }
}

// Usage
const breaker = new CircuitBreaker(5, 60000);

async function makeApiCall() {
  return breaker.execute(() => fetch('/api/data'));
}
```

## Best Practices

1. **Avoid Shared Mutable State**: Use immutable data or proper synchronization
2. **Use Promises/Async-Await**: Cleaner than callbacks
3. **Handle Errors**: Always catch and handle errors in async code
4. **Avoid Blocking**: Keep operations non-blocking
5. **Use Worker Threads**: For CPU-intensive tasks
6. **Limit Concurrency**: Use semaphores for resource-intensive operations
7. **Test Thoroughly**: Race conditions are hard to reproduce
8. **Use Timeouts**: Prevent indefinite waiting
9. **Monitor Performance**: Track async operations
10. **Document Async Code**: Make concurrency patterns clear

## Common Pitfalls

```javascript
// 1. Forgetting await
async function bad() {
  const data = fetchData(); // Missing await!
  console.log(data); // Promise, not data
}

async function good() {
  const data = await fetchData();
  console.log(data); // Actual data
}

// 2. Sequential when parallel is possible
async function slow() {
  const user = await fetchUser();
  const posts = await fetchPosts();
  const comments = await fetchComments();
}

async function fast() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);
}

// 3. Unhandled promise rejections
async function risky() {
  fetchData(); // Fire and forget - error not handled!
}

async function safe() {
  fetchData().catch(error => console.error(error));
}
```

## Related Concepts

- Thread Safety
- Atomic Operations
- Lock-Free Programming
- Actor Model
- CSP (Communicating Sequential Processes)
- Reactive Programming
