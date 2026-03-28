# Design Patterns

A practical reference for recognizing, choosing, and applying GoF design patterns.

This document expands the original `skills/solid/references/design-patterns.md` reference into a more complete guide aligned with the concepts from *Dive Into Design Patterns*: what patterns are, why they matter, the design principles beneath them, and the 22 classic GoF patterns grouped by intent. The original codebase currently contains a shorter version focused on a subset of patterns and a warning not to force them; this rewrite keeps that spirit while broadening the catalog and decision guidance. 

---

## What Are Design Patterns?

Design patterns are **typical solutions to recurring design problems** in object-oriented software. They are not copy-paste code or frameworks. A pattern is a reusable design idea: a named way to organize responsibilities, collaboration, and extension points.

Think of a pattern as a **blueprint**, not a finished building:
- An **algorithm** gives a step-by-step procedure.
- A **pattern** gives a higher-level structure for solving a class of problems.

Patterns are useful because they:
- give teams a **shared vocabulary**;
- encode **tried and tested design trade-offs**;
- improve **flexibility, maintainability, and reuse**;
- teach you to reason about **responsibilities, variation, and coupling**.

## What a Pattern Description Usually Includes

A good pattern description usually covers:
- **Intent** — what problem it solves;
- **Motivation / Problem** — why the problem appears;
- **Structure** — the roles and their relationships;
- **Applicability** — when to use it;
- **Implementation notes** — common steps and caveats;
- **Pros / Cons** — trade-offs;
- **Relations with other patterns** — how it complements or evolves into others.

## Classification of Patterns

Patterns are commonly grouped by intent:

### Creational
Focus on object creation and object setup.

Use when construction logic is complex, variable, or should be decoupled from usage.

Patterns:
- Factory Method
- Abstract Factory
- Builder
- Prototype
- Singleton

### Structural
Focus on composing classes and objects into larger structures.

Use when you need compatibility, layering, wrapping, simplification, or tree structures.

Patterns:
- Adapter
- Bridge
- Composite
- Decorator
- Facade
- Flyweight
- Proxy

### Behavioral
Focus on communication, algorithms, control flow, and assignment of responsibilities.

Use when behavior changes at runtime, work must be coordinated, or control should be decoupled.

Patterns:
- Chain of Responsibility
- Command
- Iterator
- Mediator
- Memento
- Observer
- State
- Strategy
- Template Method
- Visitor

---

## Before Patterns: Design Principles They Build On

Most patterns are concrete applications of a smaller set of design principles.

### Encapsulate What Varies

Identify what changes and isolate it from what stays stable.

Use this principle to:
- extract volatile rules into separate modules;
- localize changes;
- reduce ripple effects across the codebase.

Typical examples:
- tax calculation extracted from order total calculation;
- shipping rules extracted from order logic;
- rendering or persistence isolated from domain logic.

### Program to an Interface, Not an Implementation

Depend on abstractions instead of concrete classes.

Use this principle when:
- you want to swap implementations;
- client code should remain stable while details vary;
- you want polymorphism instead of type conditionals.

This principle sits behind patterns such as Strategy, Factory Method, Abstract Factory, Bridge, Proxy, Decorator, and Observer.

### Favor Composition Over Inheritance

Prefer delegating behavior to composed objects instead of pushing everything into a class hierarchy.

Use composition when:
- behavior has multiple dimensions of variation;
- inheritance would cause subclass explosion;
- behavior should change at runtime.

This principle is central to Strategy, Decorator, Bridge, Composite, Proxy, Adapter, and State.

### SOLID and Patterns

Patterns often help implement SOLID, but they are not substitutes for judgment.

- **SRP** — patterns split responsibilities across focused collaborators.
- **OCP** — patterns create extension points without modifying stable client code.
- **LSP** — patterns rely on substitutable abstractions.
- **ISP** — patterns work best with small, focused interfaces.
- **DIP** — patterns decouple high-level policies from low-level details.

---

## Relation Strengths: Dependency to Composition

When discussing patterns, it helps to distinguish common object relationships:

- **Dependency** — one class is affected by another’s changes.
- **Association** — one object knows or uses another.
- **Aggregation** — a whole contains parts that can outlive it.
- **Composition** — a whole owns parts and manages their lifecycle.
- **Implementation** — a class fulfills an interface contract.
- **Inheritance** — a subclass extends a superclass.

Patterns often combine these deliberately:
- Decorator and Proxy rely on **composition**.
- Template Method relies on **inheritance**.
- Strategy relies on **association/composition plus interface-based polymorphism**.
- Adapter can use **composition** or **inheritance**, though composition is usually safer.

---

## How to Use Patterns Well

### Do Use Patterns When

1. You recognize a recurring problem.
2. The pattern clearly reduces coupling or complexity.
3. The team understands the idea or can learn it easily.
4. The pattern fits an actual need, not a hypothetical one.

### Do Not Force Patterns

> Let patterns emerge from pressure in the code, not from fashion.

Bad reasons to apply a pattern:
- “It looks more senior.”
- “We may need it someday.”
- “The GoF book had a chapter about it.”
- “We always use this pattern.”

Good reasons:
- direct constructor calls are spreading everywhere;
- you keep adding type switches;
- you need runtime-swappable behavior;
- a class is growing in too many directions;
- client code is tightly coupled to infrastructure or 3rd-party APIs.

### A Good Rule of Thumb

Use the **simplest design that passes the tests and expresses intent**.
Apply patterns to remove real pain:
- duplication of construction logic;
- unstable dependencies;
- rigid inheritance hierarchies;
- branching on type;
- cross-cutting wrappers;
- hard-to-test orchestration.

---

## Creational Patterns

## 1. Factory Method

**Intent:** Define an interface for creating objects in a superclass, while allowing subclasses to decide which concrete product to create.

**Use when:**
- a class uses products but should not know their concrete types;
- frameworks need extension points for creating internal collaborators;
- construction logic is varying across subclasses.

**Structure:**
- `Creator` declares `factoryMethod()`;
- `ConcreteCreator` overrides it;
- `Product` is the abstraction;
- `ConcreteProduct` implements it.

**Benefits:**
- reduces coupling between creator and products;
- supports OCP by adding new product types via new creators;
- localizes creation logic.

**Costs:**
- introduces more subclasses;
- can become over-engineered when a simple constructor is enough.

**Typical signs:**
- subclass-specific creation decisions;
- a method that keeps doing `new SomeType()` based on context.

**Related patterns:** Abstract Factory, Template Method, Prototype.

## 2. Abstract Factory

**Intent:** Provide an interface for creating families of related objects without specifying their concrete classes.

**Use when:**
- products must be used together consistently;
- the app supports multiple product families or variants;
- you want client code independent from concrete product classes.

**Structure:**
- `AbstractFactory` declares creation methods for each product kind;
- `ConcreteFactory` produces one compatible family;
- abstract product interfaces hide concrete variants.

**Benefits:**
- guarantees family compatibility;
- centralizes product-family creation;
- cleanly supports adding a new family/variant.

**Costs:**
- harder to add a new product kind because every factory changes;
- introduces several interfaces and classes.

**Typical signs:**
- theme/platform/brand-specific object families;
- Windows vs Mac widgets, SQL vs in-memory repositories, provider-specific integrations.

**Related patterns:** Factory Method, Builder, Bridge, Facade, Singleton.

## 3. Builder

**Intent:** Construct complex objects step by step and allow different representations to be built using the same process.

**Use when:**
- constructors become telescoping or unreadable;
- object assembly has optional steps;
- the same build process can create different representations.

**Structure:**
- `Builder` declares construction steps;
- `ConcreteBuilder` implements them;
- `Director` optionally defines standard build sequences;
- `Product` is the result.

**Benefits:**
- avoids giant constructors;
- supports stepwise creation and optional configuration;
- isolates assembly logic from product logic.

**Costs:**
- more moving parts;
- can be too much for simple objects.

**Typical signs:**
- many optional parameters;
- assembling trees, documents, requests, domain aggregates, test data.

**Related patterns:** Abstract Factory, Composite, Bridge.

## 4. Prototype

**Intent:** Create new objects by copying existing ones without depending on their concrete classes.

**Use when:**
- copying is easier than constructing from scratch;
- the client only knows the abstraction of the object;
- you need preset configurations without subclass explosion.

**Structure:**
- `Prototype` exposes `clone()`;
- concrete prototypes copy themselves;
- optionally, a registry stores reusable prototypes.

**Benefits:**
- avoids tight coupling to concrete constructors;
- reduces repeated initialization logic;
- useful for preconfigured objects.

**Costs:**
- deep cloning is tricky;
- circular references and external resources complicate copies.

**Typical signs:**
- configuration presets;
- cloning shapes, document templates, editor states, command history snapshots.

**Related patterns:** Memento, Composite, Decorator, Abstract Factory.

## 5. Singleton

**Intent:** Ensure a class has only one instance and provide a global access point to it.

**Use when:**
- there must be exactly one shared instance;
- lifecycle and access must be centrally controlled.

**Use sparingly.** In many cases dependency injection or explicit composition is a better choice.

**Benefits:**
- controlled single instance;
- lazy initialization possible;
- convenient global access.

**Costs:**
- violates SRP by combining lifecycle control with business logic;
- hides dependencies like global state;
- makes unit testing and parallelism harder;
- often masks poor design.

**Typical signs:**
- process-wide configuration, registry, or gateway — but consider DI first.

**Related patterns:** Facade, Flyweight, Abstract Factory/Builder/Prototype implementations.

---

## Structural Patterns

## 6. Adapter

**Intent:** Convert one interface into another that clients expect, allowing incompatible classes to collaborate.

**Use when:**
- integrating legacy or 3rd-party code;
- you cannot change the existing service interface;
- the mismatch is at the boundary, not in domain behavior.

**Structure:**
- `Client` expects `Target` interface;
- `Adaptee` has an incompatible API;
- `Adapter` implements `Target` and delegates to `Adaptee`.

**Benefits:**
- isolates incompatibility;
- keeps the rest of the code clean;
- protects domain code from external APIs.

**Costs:**
- adds translation layer complexity;
- can hide poor integration boundaries if overused.

**Typical signs:**
- XML-to-JSON conversion;
- wrapping browser, framework, SDK, or legacy APIs.

**Related patterns:** Facade, Bridge, Decorator, Proxy.

## 7. Bridge

**Intent:** Split a large abstraction from its implementation so both can vary independently.

**Use when:**
- a class varies across two independent dimensions;
- inheritance would create combinatorial explosion;
- abstractions must switch implementations at runtime.

**Structure:**
- `Abstraction` holds a reference to `Implementation`;
- `RefinedAbstraction` extends high-level behavior;
- `ConcreteImplementation` varies low-level behavior.

**Benefits:**
- prevents subclass explosion;
- respects composition over inheritance;
- separates high-level policy from low-level mechanism.

**Costs:**
- increases indirection;
- can feel abstract until variation becomes real.

**Typical signs:**
- transport + engine;
- UI control + platform renderer;
- notification + delivery backend.

**Related patterns:** Abstract Factory, Strategy.

## 8. Composite

**Intent:** Compose objects into tree structures and treat individual objects and compositions uniformly.

**Use when:**
- you model hierarchies or recursive part-whole structures;
- leaf and container operations should share an interface.

**Structure:**
- `Component` defines common operations;
- `Leaf` handles primitive work;
- `Composite` stores children and delegates recursively.

**Benefits:**
- makes tree traversal and client code simpler;
- supports recursive composition naturally;
- good fit for UI trees, file systems, organization charts.

**Costs:**
- common interface can become too generic;
- constraints on valid children may be harder to enforce.

**Typical signs:**
- folder/file, menu/menu item, product bundle/item, DOM-like structures.

**Related patterns:** Builder, Iterator, Visitor, Flyweight.

## 9. Decorator

**Intent:** Attach new responsibilities to objects dynamically by wrapping them.

**Use when:**
- behavior should be stacked or combined flexibly;
- subclassing would explode combinations;
- wrappers should preserve the same interface.

**Structure:**
- `Component` declares interface;
- `ConcreteComponent` is the base behavior;
- `Decorator` holds another `Component`;
- `ConcreteDecorator` adds behavior before/after delegating.

**Benefits:**
- flexible alternative to inheritance;
- supports runtime composition of features;
- follows OCP.

**Costs:**
- many small wrapper classes;
- debugging wrapper chains can be harder.

**Typical signs:**
- adding caching, logging, retries, metrics, compression, encryption.

**Related patterns:** Composite, Strategy, Proxy.

## 10. Facade

**Intent:** Provide a simplified interface to a complex subsystem.

**Use when:**
- a subsystem has too many entry points;
- you want to expose only the common use cases;
- client code should stay insulated from subsystem complexity.

**Structure:**
- `Facade` delegates to multiple subsystem classes;
- clients call the facade instead of orchestrating details directly.

**Benefits:**
- lowers cognitive load;
- reduces direct coupling to subsystem classes;
- good boundary object for use cases.

**Costs:**
- can turn into a god object if it absorbs too much logic;
- may hide useful subsystem flexibility from advanced clients.

**Typical signs:**
- simplified app services over several infrastructure classes;
- SDK clients;
- “one-call” setup APIs.

**Related patterns:** Adapter, Abstract Factory, Singleton, Mediator.

## 11. Flyweight

**Intent:** Share intrinsic state between many small objects to reduce memory usage.

**Use when:**
- you have huge numbers of similar objects;
- most state can be shared;
- extrinsic state can be supplied from outside.

**Structure:**
- `Flyweight` stores shared intrinsic state;
- clients supply contextual extrinsic state;
- a factory manages flyweight reuse.

**Benefits:**
- lowers memory footprint dramatically;
- centralizes shared-state reuse.

**Costs:**
- separates intrinsic and extrinsic state, which complicates logic;
- only worth it when object count is very large.

**Typical signs:**
- text editors, game particles, map tiles, repeated metadata objects.

**Related patterns:** Singleton, Composite.

## 12. Proxy

**Intent:** Provide a placeholder or stand-in for another object to control access to it.

**Use when:**
- you need lazy loading, access control, remote access, logging, or caching;
- the client should see the same interface as the real object.

**Structure:**
- `Subject` is the common interface;
- `RealSubject` does real work;
- `Proxy` holds or retrieves the real subject and adds control logic.

**Benefits:**
- transparent access control;
- good for expensive resources;
- can add cross-cutting behaviors without changing the real subject.

**Costs:**
- may add latency or indirection;
- behavior becomes less obvious when many proxies stack together.

**Typical signs:**
- image/document lazy loading, remote service stubs, caching wrappers.

**Related patterns:** Decorator, Adapter, Facade.

---

## Behavioral Patterns

## 13. Chain of Responsibility

**Intent:** Pass a request through a chain of handlers until one handles it.

**Use when:**
- multiple handlers may process a request;
- sender should not know which receiver handles it;
- order of processing should be configurable.

**Structure:**
- `Handler` defines `setNext` and `handle`;
- concrete handlers either process or delegate onward.

**Benefits:**
- decouples sender from receiver;
- supports pipelines and middleware;
- easy to reorder or extend handlers.

**Costs:**
- request may go unhandled;
- tracing flow across long chains can be harder.

**Typical signs:**
- HTTP middleware, validation pipelines, event filters, approval workflows.

**Related patterns:** Decorator, Command.

## 14. Command

**Intent:** Encapsulate a request as an object so it can be parameterized, queued, logged, or undone.

**Use when:**
- you need undo/redo;
- actions should be queued or scheduled;
- UI senders should be decoupled from receivers.

**Structure:**
- `Command` defines `execute()`;
- optional `undo()`;
- `Invoker` triggers commands;
- `Receiver` performs actual work.

**Benefits:**
- turns behavior into first-class objects;
- supports history, retries, and audit trails;
- good fit for Tell, Don’t Ask.

**Costs:**
- introduces many command classes;
- overkill for very simple direct calls.

**Typical signs:**
- editor actions, menu items, jobs, workflows.

**Related patterns:** Memento, Prototype, Chain of Responsibility.

## 15. Iterator

**Intent:** Provide a way to traverse elements of a collection without exposing its internal representation.

**Use when:**
- collections have multiple traversal strategies;
- traversal logic should be separated from the collection;
- client code should not depend on internal structure.

**Structure:**
- `Iterator` exposes traversal operations;
- `Aggregate` creates iterators;
- concrete iterators track traversal state.

**Benefits:**
- simplifies client traversal code;
- allows multiple traversal strategies;
- respects encapsulation.

**Costs:**
- can be unnecessary in languages with strong built-in iteration constructs;
- adds objects and state to manage.

**Typical signs:**
- custom trees, graphs, paginated APIs, filtered traversals.

**Related patterns:** Factory Method, Composite.

## 16. Mediator

**Intent:** Centralize complex communications and control logic between related objects.

**Use when:**
- objects are heavily interconnected;
- communication logic is scattered across peers;
- you want to reduce many-to-many dependencies.

**Structure:**
- colleagues communicate through `Mediator` instead of directly;
- mediator coordinates workflow.

**Benefits:**
- reduces peer coupling;
- keeps interaction logic in one place;
- useful in forms, dialogs, workflows.

**Costs:**
- mediator can become a god object;
- over-centralization can hide natural domain boundaries.

**Typical signs:**
- UI dialog coordination, chat rooms, orchestration hubs.

**Related patterns:** Facade, Observer.

## 17. Memento

**Intent:** Capture and restore an object’s internal state without violating encapsulation.

**Use when:**
- you need snapshots for undo, history, or rollback;
- object internals should remain hidden from caretakers.

**Structure:**
- `Originator` creates and restores mementos;
- `Memento` stores state;
- `Caretaker` keeps history.

**Benefits:**
- supports rollback while preserving encapsulation;
- clean undo/history model.

**Costs:**
- memory-heavy with frequent snapshots;
- large object graphs make snapshots expensive.

**Typical signs:**
- editors, transactional workflows, rollback support.

**Related patterns:** Command, Prototype.

## 18. Observer

**Intent:** Define a one-to-many dependency so that when one object changes state, all dependents are notified automatically.

**Use when:**
- state changes must fan out to multiple listeners;
- publishers should not know concrete subscribers;
- you need event-driven or reactive flows.

**Structure:**
- `Subject` manages subscribers;
- `Observer` exposes update contract;
- concrete observers react to notifications.

**Benefits:**
- decouples publishers from subscribers;
- supports event-driven designs;
- easy to add new listeners.

**Costs:**
- update order may matter and become subtle;
- careless use can create event storms or hidden dependencies.

**Typical signs:**
- domain events, UI events, notifications, cache invalidation listeners.

**Related patterns:** Mediator, Strategy, Command.

## 19. State

**Intent:** Let an object alter its behavior when its internal state changes, appearing to change its class.

**Use when:**
- behavior depends heavily on state;
- code is full of large conditional branches on status;
- valid transitions matter.

**Structure:**
- `Context` delegates behavior to current `State`;
- concrete states implement state-specific behavior and transitions.

**Benefits:**
- replaces large state conditionals with polymorphism;
- localizes behavior per state;
- clarifies transitions.

**Costs:**
- adds classes for each state;
- can feel heavy for a tiny state machine.

**Typical signs:**
- order lifecycle, workflow steps, connection state, UI modes.

**Related patterns:** Strategy, Singleton.

## 20. Strategy

**Intent:** Define a family of algorithms, encapsulate each one, and make them interchangeable.

**Use when:**
- there are multiple ways to do the same thing;
- behavior should be selected at runtime;
- you want to replace type-based conditionals.

**Structure:**
- `Context` holds a `Strategy` abstraction;
- concrete strategies implement the algorithm.

**Benefits:**
- easy runtime substitution;
- cleaner than conditionals for algorithm families;
- aligns well with OCP and DIP.

**Costs:**
- more classes or functions to manage;
- clients may need awareness of available strategies.

**Typical signs:**
- pricing, routing, sorting, validation, serialization, shipping.

**Related patterns:** State, Bridge, Decorator.

## 21. Template Method

**Intent:** Define the skeleton of an algorithm in a base class while letting subclasses redefine some steps.

**Use when:**
- an algorithm is stable overall but some steps vary;
- reuse through inheritance is acceptable;
- sequence should remain controlled.

**Structure:**
- base class defines template method;
- primitive operations are abstract or overridable;
- hooks may provide optional extension points.

**Benefits:**
- reuses invariant workflow;
- standardizes algorithm sequence;
- makes extension points explicit.

**Costs:**
- based on inheritance, so less flexible than composition;
- subclasses can become tightly coupled to base algorithm design.

**Typical signs:**
- exporters, parsers, test runners, framework lifecycle methods.

**Related patterns:** Factory Method, Strategy.

## 22. Visitor

**Intent:** Separate algorithms from the objects on which they operate by moving operations into visitor objects.

**Use when:**
- you need to add many unrelated operations across a stable object structure;
- object hierarchy is stable, but operations vary frequently;
- double dispatch is useful.

**Structure:**
- `Visitor` declares a visit method per concrete element;
- `Element` declares `accept(visitor)`;
- concrete visitors implement operations.

**Benefits:**
- easy to add new operations;
- keeps element classes focused on core responsibilities;
- works well over complex object structures.

**Costs:**
- hard to add new element types because all visitors change;
- can be awkward in languages without good overload support.

**Typical signs:**
- AST processing, report generation, validation passes, export passes.

**Related patterns:** Composite, Iterator.

---

## Pattern Selection Guide

### If the main pain is object creation

- **Need subclass-controlled creation?** → Factory Method
- **Need compatible families?** → Abstract Factory
- **Need step-by-step assembly?** → Builder
- **Need copies of configured objects?** → Prototype
- **Need exactly one shared instance?** → Singleton, but verify DI would not be better

### If the main pain is structure or coupling

- **Incompatible interfaces?** → Adapter
- **Two variation dimensions?** → Bridge
- **Tree structure?** → Composite
- **Add behavior by wrapping?** → Decorator
- **Need a simpler entry point?** → Facade
- **Need memory sharing at scale?** → Flyweight
- **Need controlled access/lazy loading?** → Proxy

### If the main pain is behavior or orchestration

- **Pipeline of handlers?** → Chain of Responsibility
- **Actions as objects / undo / queue?** → Command
- **Traversal abstraction?** → Iterator
- **Many objects chatting too much?** → Mediator
- **Need snapshots/rollback?** → Memento
- **Publish/subscribe notifications?** → Observer
- **Behavior changes by status?** → State
- **Swap algorithms at runtime?** → Strategy
- **Fixed algorithm with varying steps?** → Template Method
- **Many operations over stable structures?** → Visitor

---

## Common Refactoring Triggers That Suggest Patterns

| Smell / Pressure | Pattern Often Considered |
|---|---|
| Large `switch` / `if` by type | Strategy, State, Factory Method |
| Constructor explosion | Builder |
| Incompatible 3rd-party API | Adapter |
| Subclass explosion across multiple axes | Bridge, Strategy, Decorator |
| Repeated wrappers for logging/caching/auth | Decorator, Proxy |
| Complex subsystem initialization | Facade, Abstract Factory |
| Need undo/history | Command, Memento, Prototype |
| Too many objects referencing each other | Mediator, Observer |
| Tree-like recursive structure | Composite, Visitor, Iterator |

---

## Anti-Patterns and Misuse Warnings

### Golden Hammer
Using the same pattern everywhere because the team likes it.

**Fix:** Match the pattern to the problem.

### Premature Abstraction
Adding patterns before variation exists.

**Fix:** Wait for real pressure. Prefer duplication over the wrong abstraction.

### God Facade / God Mediator
Central object grows into a dumping ground.

**Fix:** split by use case or responsibility.

### Singleton as Hidden Global State
Everything reaches into one global object.

**Fix:** prefer explicit dependencies and composition roots.

### Inheritance Abuse
Template Method or base classes become too rigid.

**Fix:** prefer Strategy, Bridge, Decorator, or composition.

---

## A Simple Pattern Review Checklist

Before introducing a pattern, ask:

1. What exact code smell or change pressure am I addressing?
2. Is the pattern reducing coupling or just adding ceremony?
3. Can a simpler refactoring solve the problem first?
4. Will the team recognize and maintain this design?
5. Does this improve testability and extension?
6. Am I introducing the pattern for a real need or a hypothetical one?

After introducing it, verify:

1. The code is easier to change.
2. Dependencies are clearer.
3. Tests became easier, not harder.
4. Names reveal intent.
5. The abstraction is earning its keep.

---

## Pattern Awareness Lens

When analyzing unfamiliar code, ask:

1. **What problem is being solved?** Creation, structure, or behavior?
2. **What varies?** Type, family, algorithm, workflow, integration, lifecycle?
3. **Where is polymorphism introduced?** Interface, subclass, composition, wrapper?
4. **What trade-off is being made?** Less coupling, more classes? More flexibility, more indirection?
5. **What would break if I added one more variant?** The answer often reveals the missing pattern.

---

## Final Guidance

Patterns are tools, not goals.

Use them to:
- isolate volatility;
- improve substitutability;
- reduce direct coupling;
- express responsibilities clearly;
- make change safer.

Avoid them when:
- a simpler design already communicates intent;
- the pattern adds more indirection than value;
- you are designing for imaginary future requirements.

> The best pattern is the one that removes real friction while keeping the code understandable.
