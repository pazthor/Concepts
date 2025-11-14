# Microservices Architecture

## Overview

Microservices architecture structures an application as a collection of small, autonomous services that are independently deployable and scalable. Each service focuses on a specific business capability.

## Structure

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   User      │     │   Order     │     │   Payment   │
│   Service   │────▶│   Service   │────▶│   Service   │
│   (DB)      │     │   (DB)      │     │   (DB)      │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │
       └───────────────────┴───────────────────┘
                           │
                   ┌───────▼────────┐
                   │   API Gateway   │
                   └────────────────┘
```

## Key Principles

1. **Single Responsibility**: Each service has one focused business capability
2. **Autonomy**: Services can be developed, deployed, and scaled independently
3. **Decentralization**: Distributed data management and governance
4. **Resilience**: Failure in one service doesn't bring down the entire system

## Core Characteristics

### Independent Deployment
- Each service can be deployed without affecting others
- Enables continuous delivery and deployment
- Reduces risk of system-wide failures

### Technology Diversity
- Different services can use different tech stacks
- Choose the best tool for each job
- Freedom to evolve technology choices

### Organized Around Business Capabilities
- Services align with business domains
- Teams own services end-to-end
- Clear boundaries and responsibilities

### Smart Endpoints, Dumb Pipes
- Business logic in services, not in middleware
- Simple communication protocols (HTTP/REST, messaging)
- Services handle their own complexity

## Benefits

- **Scalability**: Scale individual services independently
- **Flexibility**: Use different technologies per service
- **Resilience**: Isolated failures don't cascade
- **Team autonomy**: Small teams own entire services
- **Faster development**: Parallel development of services
- **Easy to understand**: Smaller, focused codebases

## Challenges

- **Complexity**: Distributed systems are inherently complex
- **Network latency**: Inter-service communication overhead
- **Data consistency**: Maintaining consistency across services
- **Testing**: End-to-end testing is more difficult
- **Deployment**: Requires sophisticated DevOps practices
- **Monitoring**: Need distributed tracing and logging

## Communication Patterns

### Synchronous (HTTP/REST, gRPC)
```
Service A ──[HTTP Request]──▶ Service B
         ◀──[HTTP Response]──
```

### Asynchronous (Message Queue)
```
Service A ──[Publish Event]──▶ Message Queue ──▶ Service B
```

## When to Use

- Large, complex applications
- Systems requiring high scalability
- Organizations with multiple development teams
- Systems with diverse technology requirements
- Applications needing frequent updates

## When NOT to Use

- Small applications
- Teams without DevOps expertise
- Systems requiring strong consistency
- Projects with tight budgets or timelines

## Example Service

```javascript
// User Service
class UserService {
  constructor(userRepository, eventBus) {
    this.userRepository = userRepository;
    this.eventBus = eventBus;
  }

  async createUser(userData) {
    const user = await this.userRepository.create(userData);

    // Publish event for other services
    await this.eventBus.publish('user.created', {
      userId: user.id,
      email: user.email
    });

    return user;
  }

  async getUser(id) {
    return this.userRepository.findById(id);
  }
}

// API Endpoint
app.post('/users', async (req, res) => {
  const user = await userService.createUser(req.body);
  res.status(201).json(user);
});

// Order Service (listens to user.created events)
eventBus.subscribe('user.created', async (event) => {
  await orderRepository.createUserProfile(event.userId);
});
```

## Best Practices

1. **Start with a monolith**: Don't start with microservices
2. **Define clear boundaries**: Use Domain-Driven Design
3. **Design for failure**: Implement circuit breakers, retries
4. **Decentralize data**: Each service owns its database
5. **API Gateway**: Single entry point for clients
6. **Service discovery**: Dynamic service location
7. **Centralized logging**: Aggregate logs from all services
8. **Distributed tracing**: Track requests across services
9. **Automate everything**: CI/CD, testing, deployment
10. **Monitor everything**: Metrics, health checks, alerts

## Related Patterns

- API Gateway
- Service Mesh
- Circuit Breaker
- Event Sourcing
- CQRS (Command Query Responsibility Segregation)
- Saga Pattern (distributed transactions)
