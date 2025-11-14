# Type Systems

## Overview

A type system is a set of rules that assigns types to programming constructs and checks that these types are used correctly. Types help catch errors, document code, and enable tooling support.

## Type Classification

### Static vs Dynamic Typing

#### Static Typing
Types are checked at compile time.

```typescript
// TypeScript (Static)
function add(a: number, b: number): number {
  return a + b;
}

add(5, 10);     // OK
add("5", "10"); // Error: Types don't match
```

**Languages**: TypeScript, Java, C++, Rust, Go

**Benefits**:
- Catch errors early (compile time)
- Better IDE support (autocomplete, refactoring)
- Self-documenting code
- Better performance (compiler optimizations)

**Drawbacks**:
- More verbose
- Longer development time
- Less flexible

#### Dynamic Typing
Types are checked at runtime.

```javascript
// JavaScript (Dynamic)
function add(a, b) {
  return a + b;
}

add(5, 10);     // 15
add("5", "10"); // "510" (string concatenation)
```

**Languages**: JavaScript, Python, Ruby, PHP

**Benefits**:
- Faster to write
- More flexible
- Less boilerplate

**Drawbacks**:
- Errors caught at runtime
- Less IDE support
- Harder to refactor

### Strong vs Weak Typing

#### Strong Typing
Strict type rules, minimal implicit conversions.

```python
# Python (Strong)
x = 5
y = "10"
result = x + y  # TypeError: unsupported operand type(s)
```

#### Weak Typing
Permissive type rules, automatic conversions.

```javascript
// JavaScript (Weak)
const x = 5;
const y = "10";
const result = x + y; // "510" (number coerced to string)
```

## Type Annotations

### TypeScript Example

```typescript
// Basic types
let name: string = "Alice";
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Arrays
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["Alice", "Bob"];

// Tuples
let person: [string, number] = ["Alice", 30];

// Enums
enum Status {
  Active,
  Inactive,
  Pending
}
let userStatus: Status = Status.Active;

// Any (escape hatch - avoid when possible)
let dynamic: any = "string";
dynamic = 42; // OK

// Unknown (safer than any)
let uncertain: unknown = "string";
// uncertain.toUpperCase(); // Error
if (typeof uncertain === "string") {
  uncertain.toUpperCase(); // OK after type guard
}

// Never (functions that never return)
function throwError(message: string): never {
  throw new Error(message);
}

// Void (no return value)
function log(message: string): void {
  console.log(message);
}
```

## Type Inference

Automatic type detection by the compiler.

```typescript
// Type inference
let inferredString = "hello"; // inferred as string
let inferredNumber = 42;      // inferred as number

// Function return type inference
function multiply(a: number, b: number) {
  return a * b; // return type inferred as number
}

// Complex inference
const users = [
  { name: "Alice", age: 30 },
  { name: "Bob", age: 25 }
]; // inferred as { name: string; age: number }[]
```

## Advanced Types

### Union Types

Multiple possible types.

```typescript
function formatId(id: string | number): string {
  if (typeof id === "string") {
    return id.toUpperCase();
  }
  return id.toString();
}

formatId("abc123"); // OK
formatId(123);      // OK
// formatId(true);  // Error
```

### Intersection Types

Combine multiple types.

```typescript
interface Person {
  name: string;
  age: number;
}

interface Employee {
  employeeId: string;
  department: string;
}

type PersonEmployee = Person & Employee;

const employee: PersonEmployee = {
  name: "Alice",
  age: 30,
  employeeId: "E123",
  department: "Engineering"
};
```

### Type Aliases

Custom type names.

```typescript
type ID = string | number;
type Point = { x: number; y: number };
type Callback = (data: string) => void;

const userId: ID = "user123";
const point: Point = { x: 10, y: 20 };
const handler: Callback = (data) => console.log(data);
```

### Literal Types

Specific values as types.

```typescript
type Direction = "north" | "south" | "east" | "west";
type Status = 200 | 404 | 500;
type Yes = true;

function move(direction: Direction) {
  console.log(`Moving ${direction}`);
}

move("north"); // OK
// move("up"); // Error
```

## Interfaces

Define object shapes.

```typescript
interface User {
  id: string;
  name: string;
  age: number;
  email?: string; // Optional property
  readonly createdAt: Date; // Read-only property
}

const user: User = {
  id: "123",
  name: "Alice",
  age: 30,
  createdAt: new Date()
};

// user.createdAt = new Date(); // Error: read-only

// Interface extension
interface Admin extends User {
  permissions: string[];
}

const admin: Admin = {
  id: "456",
  name: "Bob",
  age: 35,
  createdAt: new Date(),
  permissions: ["read", "write", "delete"]
};
```

## Generics

Type parameters for reusable code.

```typescript
// Generic function
function identity<T>(value: T): T {
  return value;
}

const numResult = identity<number>(42);
const strResult = identity<string>("hello");
const inferredResult = identity(true); // Type inferred

// Generic interface
interface Box<T> {
  value: T;
}

const numberBox: Box<number> = { value: 42 };
const stringBox: Box<string> = { value: "hello" };

// Generic constraints
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): void {
  console.log(item.length);
}

logLength("hello");     // OK (string has length)
logLength([1, 2, 3]);   // OK (array has length)
// logLength(42);       // Error (number doesn't have length)

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const result = pair("age", 30); // [string, number]
```

## Type Guards

Runtime type checking.

```typescript
// typeof guard
function processValue(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}

// instanceof guard
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}

// Custom type guard
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim();
  } else {
    pet.fly();
  }
}

// Discriminated unions
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number }
  | { kind: "square"; size: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "square":
      return shape.size ** 2;
  }
}
```

## Utility Types

Built-in type transformations.

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

// Partial<T> - all properties optional
type PartialUser = Partial<User>;
const update: PartialUser = { name: "Alice" };

// Required<T> - all properties required
interface OptionalUser {
  id?: string;
  name?: string;
}
type RequiredUser = Required<OptionalUser>;

// Readonly<T> - all properties read-only
type ReadonlyUser = Readonly<User>;
const user: ReadonlyUser = {
  id: "123",
  name: "Alice",
  email: "alice@example.com",
  age: 30
};
// user.name = "Bob"; // Error

// Pick<T, K> - select specific properties
type UserPreview = Pick<User, "id" | "name">;
const preview: UserPreview = { id: "123", name: "Alice" };

// Omit<T, K> - exclude specific properties
type UserWithoutEmail = Omit<User, "email">;
const noEmail: UserWithoutEmail = {
  id: "123",
  name: "Alice",
  age: 30
};

// Record<K, T> - object type with specific keys
type Roles = "admin" | "user" | "guest";
type Permissions = Record<Roles, string[]>;
const permissions: Permissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"]
};

// ReturnType<T> - extract return type of function
function getUser() {
  return { id: "123", name: "Alice" };
}
type UserReturnType = ReturnType<typeof getUser>;

// Parameters<T> - extract parameter types
function createUser(name: string, age: number) {}
type CreateUserParams = Parameters<typeof createUser>; // [string, number]
```

## Nominal vs Structural Typing

### Structural Typing (TypeScript)
Compatibility based on structure.

```typescript
interface Point2D {
  x: number;
  y: number;
}

interface Vector2D {
  x: number;
  y: number;
}

const point: Point2D = { x: 0, y: 0 };
const vector: Vector2D = point; // OK - same structure
```

### Nominal Typing (e.g., Java)
Compatibility based on explicit type names.

```java
// Java - nominal typing
class Point2D {
  int x;
  int y;
}

class Vector2D {
  int x;
  int y;
}

Point2D point = new Point2D();
Vector2D vector = point; // Error - different types
```

## Type Safety Benefits

```typescript
// Without types
function calculateDiscount(price, discount) {
  return price * (1 - discount);
}

calculateDiscount(100, 0.1);    // 90
calculateDiscount("100", "10"); // NaN (bug!)

// With types
function calculateDiscountSafe(
  price: number,
  discount: number
): number {
  if (discount < 0 || discount > 1) {
    throw new Error("Discount must be between 0 and 1");
  }
  return price * (1 - discount);
}

calculateDiscountSafe(100, 0.1);    // 90
// calculateDiscountSafe("100", "10"); // Compile error
```

## Best Practices

1. **Use Type Inference**: Let compiler infer when obvious
2. **Avoid `any`**: Use `unknown` for truly dynamic types
3. **Be Specific**: Use narrow types (literals, unions)
4. **Use Interfaces**: For object shapes
5. **Use Type Aliases**: For unions and complex types
6. **Use Generics**: For reusable, type-safe code
7. **Enable Strict Mode**: Catch more errors
8. **Document with Types**: Types serve as documentation
9. **Use Utility Types**: Leverage built-in helpers
10. **Type Guard Complex Types**: Use custom guards

## Type System Comparison

| Feature | Static | Dynamic | Strong | Weak |
|---------|--------|---------|--------|------|
| Type Checking | Compile time | Runtime | Strict | Permissive |
| Examples | TypeScript, Java | JavaScript, Python | Python | JavaScript |
| Error Detection | Early | Late | Explicit | Implicit |
| Flexibility | Less | More | Less | More |

## Common Type Errors

```typescript
// Type mismatch
function greet(name: string) {
  console.log(`Hello, ${name}`);
}
// greet(123); // Error

// Null/undefined errors
function getLength(str: string) {
  return str.length;
}
// getLength(null); // Error with strict null checks

// Array access
const numbers: number[] = [1, 2, 3];
// const item: number = numbers[10]; // undefined at runtime

// Better with optional
const item: number | undefined = numbers[10];
```

## Related Concepts

- Type Inference
- Type Checking
- Polymorphism
- Variance (covariance, contravariance)
- Gradual Typing
- Dependent Types
