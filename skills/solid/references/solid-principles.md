# SOLID Principles

The **SOLID principles** are five design guidelines that help developers build **maintainable, scalable, and flexible software**. They promote clean architecture and reduce code complexity over time.

---

## 1. Single Responsibility Principle (SRP)

> **"A class should have only one reason to change."**

### ✅ Key Idea

Each class or module should focus on **one responsibility**.

### ❌ Bad Example

```ts
class UserService {
  saveUser(user) { /* save logic */ }
  sendEmail(user) { /* email logic */ }
}
```

### ✅ Good Example

```ts
class UserRepository {
  save(user) { /* save logic */ }
}

class EmailService {
  send(user) { /* email logic */ }
}
```

### 💡 Benefits

* Easier to maintain
* Better testability
* Clear separation of concerns

---

## 2. Open/Closed Principle (OCP)

> **"Software entities should be open for extension, but closed for modification."**

### ✅ Key Idea

You should be able to **add new behavior without changing existing code**.

### ❌ Bad Example

```ts
function getDiscount(type) {
  if (type === "regular") return 0.1;
  if (type === "vip") return 0.2;
}
```

### ✅ Good Example

```ts
interface Discount {
  get(): number;
}

class RegularDiscount implements Discount {
  get() { return 0.1; }
}

class VipDiscount implements Discount {
  get() { return 0.2; }
}
```

### 💡 Benefits

* Avoids breaking existing code
* Encourages extensibility
* Reduces regression bugs

---

## 3. Liskov Substitution Principle (LSP)

> **"Subtypes must be substitutable for their base types."**

### ✅ Key Idea

A subclass should behave **correctly when used in place of its parent class**.

### ❌ Bad Example

```ts
class Bird {
  fly() {}
}

class Penguin extends Bird {
  fly() {
    throw new Error("Can't fly");
  }
}
```

### ✅ Good Example

```ts
class Bird {}

class FlyingBird extends Bird {
  fly() {}
}

class Penguin extends Bird {}
```

### 💡 Benefits

* Prevents unexpected behavior
* Ensures reliable inheritance
* Improves polymorphism

---

## 4. Interface Segregation Principle (ISP)

> **"Clients should not be forced to depend on interfaces they do not use."**

### ✅ Key Idea

Split large interfaces into **smaller, specific ones**.

### ❌ Bad Example

```ts
interface Worker {
  work();
  eat();
}

class Robot implements Worker {
  work() {}
  eat() {} // unnecessary
}
```

### ✅ Good Example

```ts
interface Workable {
  work();
}

interface Eatable {
  eat();
}

class Robot implements Workable {
  work() {}
}
```

### 💡 Benefits

* Reduces unnecessary dependencies
* Improves flexibility
* Cleaner abstractions

---

## 5. Dependency Inversion Principle (DIP)

> **"Depend on abstractions, not concretions."**

### ✅ Key Idea

High-level modules should not depend on low-level modules. Both should depend on **interfaces/abstractions**.

### ❌ Bad Example

```ts
class MySQLDatabase {
  connect() {}
}

class App {
  db = new MySQLDatabase();
}
```

### ✅ Good Example

```ts
interface Database {
  connect(): void;
}

class MySQLDatabase implements Database {
  connect() {}
}

class App {
  constructor(private db: Database) {}
}
```

### 💡 Benefits

* Easier to swap implementations
* Better testability (mocking)
* Loose coupling

---

## 🔗 Summary Table

| Principle | Focus          | Goal                       |
| --------- | -------------- | -------------------------- |
| SRP       | Responsibility | One reason to change       |
| OCP       | Extensibility  | Add without modifying      |
| LSP       | Substitution   | Safe inheritance           |
| ISP       | Interfaces     | Small & specific           |
| DIP       | Dependencies   | Abstractions over concrete |

---

## 🧠 Quick Heuristics

* If a class is doing too much → **SRP**
* If you keep editing old code to add features → **OCP**
* If inheritance breaks behavior → **LSP**
* If interfaces feel bloated → **ISP**
* If classes are tightly coupled → **DIP**

---

## 🚀 Final Thoughts

Applying SOLID principles leads to:

* Cleaner architecture
* Easier refactoring
* More scalable systems
* Better collaboration in teams

> SOLID is not about rigid rules — it's about **writing code that survives change**.
