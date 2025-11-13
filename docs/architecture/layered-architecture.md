# Layered Architecture

## Overview

Layered architecture is one of the most common architectural patterns. It organizes the system into horizontal layers, where each layer has a specific responsibility and interacts only with adjacent layers.

## Structure

```
┌─────────────────────────────┐
│   Presentation Layer        │  (UI, Controllers)
├─────────────────────────────┤
│   Business Logic Layer      │  (Domain logic, Services)
├─────────────────────────────┤
│   Data Access Layer         │  (Repositories, ORMs)
├─────────────────────────────┤
│   Database Layer            │  (Persistence)
└─────────────────────────────┘
```

## Key Principles

1. **Separation of Concerns**: Each layer has a distinct responsibility
2. **Dependency Rule**: Layers depend only on layers below them
3. **Abstraction**: Upper layers know lower layers through interfaces

## Typical Layers

### Presentation Layer
- Handles user interface and user interaction
- Formats data for display
- Delegates business logic to the layer below

### Business Logic Layer
- Contains core business rules and workflows
- Orchestrates application behavior
- Independent of UI and data access details

### Data Access Layer
- Manages data persistence and retrieval
- Abstracts database operations
- Provides clean API for business layer

### Database Layer
- Physical data storage
- Database management system

## Benefits

- **Easy to understand**: Clear separation makes the architecture intuitive
- **Maintainable**: Changes in one layer don't affect others
- **Testable**: Each layer can be tested independently
- **Reusable**: Business logic can be reused across different UIs

## Drawbacks

- **Performance overhead**: Each layer adds abstraction overhead
- **Tight coupling**: Can lead to tight coupling between layers
- **Changes propagation**: Simple changes might require updates across all layers

## When to Use

- Traditional web applications
- Enterprise applications with clear separation of concerns
- Systems where maintainability is prioritized over performance
- Teams with clear role separation (UI, backend, database)

## Example

```javascript
// Presentation Layer
class UserController {
  constructor(userService) {
    this.userService = userService;
  }

  async getUser(req, res) {
    const user = await this.userService.getUserById(req.params.id);
    res.json(user);
  }
}

// Business Logic Layer
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async getUserById(id) {
    const user = await this.userRepository.findById(id);
    if (!user) throw new Error('User not found');
    return this.formatUser(user);
  }

  formatUser(user) {
    // Business logic for user formatting
    return { ...user, fullName: `${user.firstName} ${user.lastName}` };
  }
}

// Data Access Layer
class UserRepository {
  constructor(database) {
    this.db = database;
  }

  async findById(id) {
    return this.db.query('SELECT * FROM users WHERE id = ?', [id]);
  }
}
```

## Best Practices

1. Keep layers loosely coupled using interfaces
2. Avoid skipping layers (don't access database from presentation)
3. Use dependency injection for layer interactions
4. Keep business logic out of presentation layer
5. Use DTOs (Data Transfer Objects) between layers
