---
name: knowledge-design-patterns
description: >-
  GoF design patterns organised by intent (creational, structural, behavioral).
  One-line descriptions, when to use, when to avoid for the most commonly useful
  patterns. Anti-patterns around pattern overuse. Use when selecting or
  recommending design patterns for implementation.
---

# Design Patterns (GoF)

The 12 most commonly useful patterns from the Gang of Four, plus guidance on when patterns cause more harm than good.

## Creational Patterns

### Factory Method
- **What**: Define an interface for creating objects; let subclasses decide which class to instantiate.
- **Use when**: Object creation logic varies by type and you want to decouple creation from usage.
- **Avoid when**: Only one type exists now and you're not expecting variation.

### Abstract Factory
- **What**: Create families of related objects without specifying concrete classes.
- **Use when**: You have multiple product families (e.g. UI themes, platform-specific components) that must stay consistent.
- **Avoid when**: You only have one product family — use Factory Method instead.

### Builder
- **What**: Construct complex objects step-by-step, separating construction from representation.
- **Use when**: Object has many optional parameters, telescoping constructors, or multiple valid configurations.
- **Avoid when**: Object is simple enough for a constructor or struct literal.

### Singleton
- **What**: Ensure a class has exactly one instance with global access.
- **Use when**: Truly global state (logger, config, connection pool) where multiple instances cause problems.
- **Avoid when**: Almost always. Prefer dependency injection. Singletons hide dependencies and hinder testability.

## Structural Patterns

### Adapter
- **What**: Convert one interface into another that clients expect.
- **Use when**: Integrating third-party libraries, legacy code, or systems with incompatible interfaces.
- **Avoid when**: You control both sides and can unify the interface directly.

### Decorator
- **What**: Attach additional behavior to an object dynamically by wrapping it.
- **Use when**: You need composable, stackable behaviors (logging, retry, caching, auth middleware).
- **Avoid when**: Behavior is static and known at compile time — direct implementation is simpler.

### Facade
- **What**: Provide a simplified interface to a complex subsystem.
- **Use when**: A subsystem has many classes and clients only need a subset of capabilities.
- **Avoid when**: The subsystem is already simple — the facade becomes an unnecessary indirection.

### Composite
- **What**: Compose objects into tree structures; treat individual objects and compositions uniformly.
- **Use when**: You have recursive/tree structures (file systems, UI component trees, org hierarchies).
- **Avoid when**: Your data is flat — forcing a tree structure adds needless complexity.

## Behavioral Patterns

### Strategy
- **What**: Define a family of algorithms, encapsulate each one, and make them interchangeable.
- **Use when**: Multiple algorithms for the same task (sorting, pricing, routing) selected at runtime.
- **Avoid when**: Only one algorithm exists. A function parameter or lambda often suffices for simple cases.

### Observer
- **What**: Define a one-to-many dependency so dependents are notified of state changes.
- **Use when**: Decoupling event producers from consumers (UI events, domain events, pub/sub).
- **Avoid when**: Only one subscriber exists — direct call is clearer. Watch for memory leaks from unremoved listeners.

### Template Method
- **What**: Define the skeleton of an algorithm; let subclasses override specific steps.
- **Use when**: Multiple variants of an algorithm share the same structure but differ in details.
- **Avoid when**: You can achieve the same with strategy/composition — prefer composition over inheritance.

### Command
- **What**: Encapsulate a request as an object, enabling parameterisation, queuing, undo, and logging.
- **Use when**: You need undo/redo, command queues, macro recording, or deferred execution.
- **Avoid when**: Simple one-shot operations where a function call suffices.

## Selection Heuristic

```
Need to create objects?
  └─ Complex construction → Builder
  └─ Family of related objects → Abstract Factory
  └─ Varying type at runtime → Factory Method

Need to compose/wrap behavior?
  └─ Stackable concerns → Decorator
  └─ Simplify subsystem → Facade
  └─ Tree structure → Composite
  └─ Adapt interface → Adapter

Need to vary behavior?
  └─ Interchangeable algorithms → Strategy
  └─ Notify dependents → Observer
  └─ Same skeleton, different steps → Template Method
  └─ Encapsulate action → Command
```

## Anti-Patterns: Pattern Overuse

- **Pattern mania** — applying patterns for their own sake. If the simple version is readable and maintainable, it's correct.
- **Speculative generality** — using Abstract Factory "in case we need another family later." YAGNI applies.
- **Singleton abuse** — global mutable state disguised as a pattern. Prefer DI.
- **Deep inheritance hierarchies** — Template Method chains 5 levels deep. Prefer composition.
- **Over-abstraction** — a Strategy interface with only one implementation. Remove it until you actually need the second one.

### Rule of Three
Don't extract a pattern until you see the **same structural problem three times**. Before that, duplication is cheaper than the wrong abstraction.

## Cross-References

- See `principles/knowledge-solid` — patterns often exist to satisfy SOLID principles (especially OCP and DIP)
- See `architecture/knowledge-clean-architecture` — patterns like Adapter and Facade appear at layer boundaries
