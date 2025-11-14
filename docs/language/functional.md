# Functional Programming

## Overview

Functional Programming (FP) is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids changing state and mutable data. It emphasizes the application of functions, immutability, and declarative code.

## Core Principles

### 1. Pure Functions

**Definition**: Functions that always return the same output for the same input and have no side effects.

```javascript
// Pure function
function add(a, b) {
  return a + b;
}

add(2, 3); // Always returns 5
add(2, 3); // Always returns 5

// Impure function (has side effects)
let total = 0;
function addToTotal(value) {
  total += value; // Modifies external state
  return total;
}

addToTotal(5); // Returns 5
addToTotal(5); // Returns 10 (different result!)
```

**Benefits of Pure Functions**:
- Predictable and testable
- Easy to reason about
- Can be cached (memoization)
- Safe for parallel execution
- No hidden dependencies

```javascript
// Pure function example
function calculateDiscount(price, discountPercent) {
  return price * (1 - discountPercent / 100);
}

// Impure - depends on external state
let discountRate = 10;
function calculateDiscountImpure(price) {
  return price * (1 - discountRate / 100);
}
```

### 2. Immutability

**Definition**: Data cannot be changed after creation. Instead, new data structures are created.

```javascript
// Mutable (bad)
const numbers = [1, 2, 3];
numbers.push(4); // Modifies original array
console.log(numbers); // [1, 2, 3, 4]

// Immutable (good)
const numbers = [1, 2, 3];
const newNumbers = [...numbers, 4]; // Creates new array
console.log(numbers);    // [1, 2, 3] (unchanged)
console.log(newNumbers); // [1, 2, 3, 4]

// Immutable object updates
const user = { name: 'John', age: 30 };
const updatedUser = { ...user, age: 31 }; // New object
console.log(user);        // { name: 'John', age: 30 }
console.log(updatedUser); // { name: 'John', age: 31 }
```

**Benefits**:
- No unexpected changes
- Easier to track data flow
- Safer for concurrent programming
- Easier debugging and testing

### 3. First-Class Functions

**Definition**: Functions are treated as values - can be assigned to variables, passed as arguments, and returned from other functions.

```javascript
// Assign function to variable
const greet = function(name) {
  return `Hello, ${name}!`;
};

// Pass function as argument
function executeFunction(fn, value) {
  return fn(value);
}

executeFunction(greet, 'Alice'); // "Hello, Alice!"

// Return function from function
function multiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiplier(2);
const triple = multiplier(3);
console.log(double(5)); // 10
console.log(triple(5)); // 15
```

### 4. Higher-Order Functions

**Definition**: Functions that take other functions as arguments or return functions.

```javascript
// map - transforms array elements
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// filter - selects elements
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]

// reduce - combines elements
const sum = numbers.reduce((acc, n) => acc + n, 0);
console.log(sum); // 15

// Custom higher-order function
function repeat(n, action) {
  for (let i = 0; i < n; i++) {
    action(i);
  }
}

repeat(3, i => console.log(`Iteration ${i}`));
// Iteration 0
// Iteration 1
// Iteration 2
```

### 5. Function Composition

**Definition**: Combining simple functions to build more complex ones.

```javascript
// Simple functions
const add = x => x + 2;
const multiply = x => x * 3;
const square = x => x * x;

// Manual composition
const result = square(multiply(add(2)));
console.log(result); // 144 ((2+2)*3)^2

// Composition helper
const compose = (...fns) => x =>
  fns.reduceRight((acc, fn) => fn(acc), x);

const calculate = compose(square, multiply, add);
console.log(calculate(2)); // 144

// Pipe (left-to-right composition)
const pipe = (...fns) => x =>
  fns.reduce((acc, fn) => fn(acc), x);

const calculate2 = pipe(add, multiply, square);
console.log(calculate2(2)); // 144
```

### 6. Recursion

**Definition**: Function calling itself to solve problems.

```javascript
// Factorial
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

console.log(factorial(5)); // 120

// Sum array
function sumArray(arr) {
  if (arr.length === 0) return 0;
  return arr[0] + sumArray(arr.slice(1));
}

console.log(sumArray([1, 2, 3, 4])); // 10

// Tail recursion (optimizable)
function factorialTail(n, accumulator = 1) {
  if (n <= 1) return accumulator;
  return factorialTail(n - 1, n * accumulator);
}

console.log(factorialTail(5)); // 120
```

## Common Functional Patterns

### Map, Filter, Reduce

```javascript
const users = [
  { name: 'Alice', age: 25, active: true },
  { name: 'Bob', age: 30, active: false },
  { name: 'Charlie', age: 35, active: true }
];

// Get names of active users
const activeNames = users
  .filter(user => user.active)
  .map(user => user.name);
// ['Alice', 'Charlie']

// Calculate total age of active users
const totalAge = users
  .filter(user => user.active)
  .reduce((sum, user) => sum + user.age, 0);
// 60

// Transform data structure
const userMap = users.reduce((map, user) => {
  map[user.name] = user.age;
  return map;
}, {});
// { Alice: 25, Bob: 30, Charlie: 35 }
```

### Currying

**Definition**: Transforming a function with multiple arguments into a sequence of functions with single arguments.

```javascript
// Regular function
function add(a, b, c) {
  return a + b + c;
}

// Curried version
function addCurried(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

console.log(addCurried(1)(2)(3)); // 6

// ES6 arrow function
const addCurriedArrow = a => b => c => a + b + c;
console.log(addCurriedArrow(1)(2)(3)); // 6

// Practical example
const multiply = a => b => a * b;
const double = multiply(2);
const triple = multiply(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15

// Generic curry function
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function(...moreArgs) {
        return curried.apply(this, args.concat(moreArgs));
      };
    }
  };
}

const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1)(2, 3)); // 6
```

### Partial Application

**Definition**: Fixing some arguments of a function to create a new function.

```javascript
function multiply(a, b, c) {
  return a * b * c;
}

// Partial application
function partial(fn, ...fixedArgs) {
  return function(...remainingArgs) {
    return fn(...fixedArgs, ...remainingArgs);
  };
}

const multiplyBy2 = partial(multiply, 2);
console.log(multiplyBy2(3, 4)); // 24

const multiplyBy2And3 = partial(multiply, 2, 3);
console.log(multiplyBy2And3(4)); // 24
```

### Memoization

**Definition**: Caching function results for performance optimization.

```javascript
function memoize(fn) {
  const cache = {};
  return function(...args) {
    const key = JSON.stringify(args);
    if (key in cache) {
      console.log('From cache');
      return cache[key];
    }
    console.log('Computing...');
    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

const fibonacci = memoize(function(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(10)); // Computing... 55
console.log(fibonacci(10)); // From cache 55
```

## Declarative vs Imperative

### Imperative (How)
```javascript
// How to do it
const numbers = [1, 2, 3, 4, 5];
const doubled = [];
for (let i = 0; i < numbers.length; i++) {
  doubled.push(numbers[i] * 2);
}
```

### Declarative (What)
```javascript
// What to do
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
```

## Practical Example

```javascript
// Imperative approach
function processOrders(orders) {
  const validOrders = [];
  for (let i = 0; i < orders.length; i++) {
    if (orders[i].status === 'pending' && orders[i].total > 0) {
      validOrders.push(orders[i]);
    }
  }

  const totals = [];
  for (let i = 0; i < validOrders.length; i++) {
    totals.push(validOrders[i].total);
  }

  let sum = 0;
  for (let i = 0; i < totals.length; i++) {
    sum += totals[i];
  }

  return sum;
}

// Functional approach
const processOrdersFP = orders =>
  orders
    .filter(order => order.status === 'pending' && order.total > 0)
    .map(order => order.total)
    .reduce((sum, total) => sum + total, 0);

// Even more functional with composition
const isPending = order => order.status === 'pending';
const hasTotal = order => order.total > 0;
const getTotal = order => order.total;
const sum = (acc, val) => acc + val;

const processOrdersComposed = orders =>
  orders
    .filter(order => isPending(order) && hasTotal(order))
    .map(getTotal)
    .reduce(sum, 0);
```

## Benefits of Functional Programming

- **Predictability**: Pure functions are predictable
- **Testability**: Easy to test pure functions
- **Concurrency**: Immutability enables safe parallelism
- **Modularity**: Small, composable functions
- **Debugging**: Easier to trace bugs
- **Readability**: Declarative code is clearer

## Challenges

- **Learning Curve**: Paradigm shift for OOP developers
- **Performance**: Immutability can impact performance
- **Debugging**: Stack traces can be complex
- **Verbosity**: Sometimes requires more code
- **Not Always Natural**: Some problems suit imperative style

## When to Use

- Data transformation pipelines
- Concurrent/parallel processing
- Complex business logic
- State management (Redux, etc.)
- Mathematical computations

## Best Practices

1. **Write Pure Functions**: Avoid side effects
2. **Embrace Immutability**: Don't mutate data
3. **Use Higher-Order Functions**: Leverage map, filter, reduce
4. **Compose Functions**: Build complex from simple
5. **Avoid Loops**: Use recursion or array methods
6. **Keep Functions Small**: Single responsibility
7. **Use Descriptive Names**: Self-documenting code
8. **Handle Errors Functionally**: Use Maybe/Either types

## Functional Programming in JavaScript

```javascript
// Point-free style (no explicit arguments)
const double = x => x * 2;
const increment = x => x + 1;
const square = x => x * x;

// Without point-free
const processNumber1 = x => square(increment(double(x)));

// With point-free (using compose)
const processNumber2 = compose(square, increment, double);

// Array operations
const numbers = [1, 2, 3, 4, 5];

// Chaining
numbers
  .filter(n => n % 2 === 0)
  .map(n => n * 2)
  .reduce((sum, n) => sum + n, 0); // 12

// Using functions
const isEven = n => n % 2 === 0;
const doubleNum = n => n * 2;
const sumNum = (sum, n) => sum + n;

numbers
  .filter(isEven)
  .map(doubleNum)
  .reduce(sumNum, 0); // 12
```

## Functional Libraries

- **Lodash/FP**: Functional utilities
- **Ramda**: Functional programming library
- **Immutable.js**: Immutable data structures
- **RxJS**: Reactive programming

## Related Concepts

- Lambda Calculus
- Category Theory
- Monads and Functors
- Reactive Programming
- Lazy Evaluation
