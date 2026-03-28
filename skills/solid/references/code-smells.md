# Code Smells & Refactoring Signals

## What Are Code Smells?

Code smells are **warning signs** that the design of the code is degrading. They are not necessarily bugs, but they usually indicate code that is getting harder to understand, extend, test, or safely change.

A smell is a prompt to ask:

- Why is this code hard to change?
- Why does this logic feel scattered, duplicated, or awkward?
- Which refactoring would reduce the friction?

Regular refactoring prevents a project from gradually slowing down under the weight of accidental complexity.

---

## How to Use This Guide

For each smell, look at four things:

1. **Signs & symptoms** — how it tends to appear in code
2. **Why it happens** — the design pressure behind it
3. **Typical refactorings** — common ways to improve it
4. **Notes / exceptions** — when not to overcorrect

---

## Smell Categories

### 1. Bloaters
Code that has grown too large to work with comfortably.

- Long Method
- Large Class
- Primitive Obsession
- Long Parameter List
- Data Clumps

### 2. Object-Orientation Abusers
Cases where object-oriented principles were applied poorly or incompletely.

- Switch Statements
- Temporary Field
- Refused Bequest
- Alternative Classes with Different Interfaces

### 3. Change Preventers
Design that makes small changes ripple through many places.

- Divergent Change
- Shotgun Surgery
- Parallel Inheritance Hierarchies

### 4. Dispensables
Things that exist in the codebase but add little or negative value.

- Comments
- Duplicate Code
- Lazy Class
- Data Class
- Dead Code
- Speculative Generality

### 5. Couplers
Excessive coupling between classes or too much delegation.

- Feature Envy
- Inappropriate Intimacy
- Message Chains
- Middle Man

### 6. Other Smells
Useful smells that do not fit neatly in the groups above.

- Incomplete Library Class

---

# 1. Bloaters

## Long Method

### Signs & Symptoms
- A method keeps growing and now mixes multiple steps
- You feel compelled to add comments to explain sections
- The method contains nested conditionals and loops
- A method longer than roughly 10 lines should at least raise suspicion

### Why It Happens
It is psychologically easier to add “just a few more lines” than to stop and extract a new method. Over time, the method becomes the dumping ground for validation, calculation, persistence, logging, and branching.

### Why It Hurts
- Harder to understand end-to-end
- Harder to test small pieces of behavior
- Easier to hide duplication
- Higher chance of unintended side effects during edits

### Typical Refactorings
- **Extract Method**
- **Replace Temp with Query**
- **Introduce Parameter Object**
- **Preserve Whole Object**
- **Replace Method with Method Object**
- **Decompose Conditional**

### Heuristics
- If you need a comment inside a method, that block probably wants a name
- If a loop or conditional represents one business step, extract it
- If local variables prevent extraction, that is a design clue, not a reason to stop

### Payoff
Shorter methods live longer, are easier to review, and are less likely to accumulate hidden duplication.

### Note
Extra small methods almost never create meaningful performance problems in modern systems.

---

## Large Class

### Signs & Symptoms
- Too many fields
- Too many methods
- Too many reasons to change
- A class is doing UI work, persistence, business rules, formatting, and notifications all together

### Why It Happens
Classes often start focused. Then each new feature gets attached to the same familiar place because adding to an existing class feels cheaper than modeling a new abstraction.

### Why It Hurts
- High cognitive load
- Low cohesion
- Changes become risky because everything is packed together
- The class becomes the natural home for even more unrelated behavior

### Typical Refactorings
- **Extract Class**
- **Extract Subclass**
- **Extract Interface**
- **Duplicate Observed Data** when UI and domain responsibilities are mixed

### Heuristics
- Ask: “What are the distinct responsibilities here?”
- Split by behavior that changes for different reasons
- Split by data that is always used together

### Payoff
Smaller classes reduce the number of facts a developer must hold in their head and often surface previously hidden duplication.

---

## Primitive Obsession

### Signs & Symptoms
- Domain concepts modeled with raw strings, ints, booleans, arrays, or constants
- Type codes like `ADMIN = 1`, `EDITOR = 2`
- Strings acting as field names or pseudo-schema keys
- Many related primitive fields with behavior scattered elsewhere

### Why It Happens
Creating a primitive field is easier than introducing a class. Over time, the “simple” field grows business rules, validation rules, formatting rules, and semantic meaning.

### Why It Hurts
- Invalid states are easy to represent
- Rules get duplicated across the codebase
- Semantic meaning remains implicit
- Refactoring becomes harder because all the logic is spread around

### Typical Refactorings
- **Replace Data Value with Object**
- **Introduce Parameter Object**
- **Preserve Whole Object**
- **Replace Type Code with Class**
- **Replace Type Code with Subclasses**
- **Replace Type Code with State/Strategy**
- **Replace Array with Object**

### Good Targets for Value Objects
- Money
- Email
- Phone number
- Date range
- User role
- Address
- Percentage / rate

### Payoff
Better type safety, clearer intent, behavior close to the data it belongs to, and easier discovery of duplication.

---

## Long Parameter List

### Signs & Symptoms
- More than 3–4 parameters regularly
- Many parameters are always passed together
- Calls become noisy or easy to misorder
- The method accepts configuration flags that control multiple behaviors

### Why It Happens
Sometimes multiple algorithms were merged into one method. Other times, developers push construction logic to callers to reduce coupling, but the resulting parameter list becomes its own form of complexity.

### Why It Hurts
- Hard to read and use correctly
- Hard to remember what each argument means
- Easy to pass incorrect values in the wrong order
- Often signals missing abstractions

### Typical Refactorings
- **Replace Parameter with Method Call**
- **Preserve Whole Object**
- **Introduce Parameter Object**

### Heuristics
- If arguments come from the same source object, consider passing the object
- If arguments travel as a bundle across several methods, model that bundle explicitly
- Beware removing parameters if it would introduce the wrong dependency

### Payoff
Shorter method signatures, more readable call sites, and clearer ownership of related data.

---

## Data Clumps

### Signs & Symptoms
- The same group of variables appears repeatedly
- The same fields are passed together from method to method
- Database credentials, coordinates, date ranges, or address parts appear as recurring clusters

### Why It Happens
Often caused by copy-paste development or weak domain modeling. Instead of naming the concept, the code repeats its parts.

### Quick Test
Delete one value from the group mentally. If the others stop making sense together, the clump probably wants to become an object.

### Typical Refactorings
- **Extract Class**
- **Introduce Parameter Object**
- **Preserve Whole Object**
- Move related behavior into the new class as well

### Payoff
Better organization, smaller signatures, and a clearer place for business rules related to that grouped data.

### Note
Passing an entire object can create unwanted coupling, so refactor with intent, not mechanically.

---

# 2. Object-Orientation Abusers

## Switch Statements

### Signs & Symptoms
- Large `switch` or `if/else` trees based on type codes or modes
- Similar conditionals duplicated in several places
- Adding one new case requires edits across multiple methods

### Why It Happens
A program starts with a small conditional and grows. New behavior gets added by appending branches instead of moving the variation into polymorphic objects.

### Why It Hurts
- Violates open/closed design
- Scatters related behavior across the codebase
- Makes extension expensive and fragile

### Typical Refactorings
- **Extract Method**
- **Move Method**
- **Replace Type Code with Subclasses**
- **Replace Type Code with State/Strategy**
- **Replace Conditional with Polymorphism**
- **Replace Parameter with Explicit Methods**
- **Introduce Null Object**

### When to Leave It Alone
- The switch is small and stable
- It is part of a straightforward factory
- Polymorphism would add more ceremony than value

### Payoff
Better organization of variant behavior and easier addition of new cases.

---

## Temporary Field

### Signs & Symptoms
- A class has fields that are only used during a specific algorithm or scenario
- Most of the time those fields are empty, null, or meaningless

### Why It Happens
To avoid a long parameter list inside one method, temporary algorithm state gets promoted to object fields even though it is not part of the object’s real lifecycle.

### Why It Hurts
- The object’s state becomes confusing
- Readers cannot tell what data is essential vs incidental
- Invariants become harder to reason about

### Typical Refactorings
- **Extract Class**
- **Replace Method with Method Object**
- **Introduce Null Object** for existence checks when appropriate

### Payoff
Cleaner objects and clearer algorithm boundaries.

---

## Refused Bequest

### Signs & Symptoms
- A subclass inherits methods or fields it does not use
- Subclass methods throw unsupported-operation exceptions
- The inheritance relationship exists mainly to reuse code

### Why It Happens
Inheritance was chosen for convenience rather than because the subtype is a real substitute for the parent.

### Why It Hurts
- Broken mental model of the hierarchy
- Violates substitutability
- Confuses maintainers about the true relationship between types

### Typical Refactorings
- **Replace Inheritance with Delegation**
- **Extract Superclass** for the genuinely shared part

### Payoff
A more honest model and clearer type relationships.

---

## Alternative Classes with Different Interfaces

### Signs & Symptoms
- Two classes do almost the same thing but expose different method names
- Teams reinvent similar abstractions in parallel

### Why It Happens
One developer often does not know that an equivalent class already exists.

### Why It Hurts
- Duplicate concepts
- Higher learning cost
- Harder substitution and reuse
- More adapters and branching in client code

### Typical Refactorings
- **Rename Method**
- **Move Method**
- **Add Parameter**
- **Parameterize Method**
- **Extract Superclass**
- Delete one class if convergence makes it redundant

### When to Leave It Alone
If the classes come from different third-party libraries, merging may not be worth the cost.

### Payoff
Less duplication and a more coherent API surface.

---

# 3. Change Preventers

## Divergent Change

### Signs & Symptoms
- One class is modified for many unrelated kinds of changes
- Adding a new product type means editing find/display/order logic all in the same class

### Why It Happens
Responsibilities that change for different reasons were packed together.

### Why It Hurts
A single class becomes the hotspot for unrelated modifications, increasing risk and merge conflicts.

### Typical Refactorings
- **Extract Class**
- **Extract Superclass**
- **Extract Subclass**

### Payoff
Better organization, less duplication, simpler support.

---

## Shotgun Surgery

### Signs & Symptoms
- One conceptual change requires tiny edits in many files or classes
- Responsibility is spread thinly across the codebase

### Why It Happens
The behavior for one concern has been fragmented, often as an overreaction to other smells.

### Why It Hurts
- High change cost
- Easy to miss one required edit
- Greater regression risk

### Typical Refactorings
- **Move Method**
- **Move Field**
- **Inline Class**
- Create a new home for the behavior if necessary

### Payoff
More localized changes, easier maintenance, less duplication.

---

## Parallel Inheritance Hierarchies

### Signs & Symptoms
- Creating a subclass in one hierarchy forces creating a matching subclass in another hierarchy
- Two hierarchies evolve in lockstep

### Why It Happens
Related behavior and data were split into separate hierarchies that now must mirror each other.

### Why It Hurts
- Duplicated structure
- Increased effort for every extension
- Architectural rigidity

### Typical Refactorings
- Make objects in one hierarchy refer to objects in the other
- Then use **Move Method** and **Move Field** to collapse one hierarchy

### When to Leave It Alone
Sometimes parallel hierarchies are the least bad option. If deduplication creates uglier code, stop.

### Payoff
Less structural duplication and potentially clearer ownership.

---

# 4. Dispensables

## Comments

### Signs & Symptoms
- Methods are filled with explanatory comments
- Comments compensate for unclear names or tangled code

### Why It Happens
The author sensed the code was hard to understand and added prose instead of improving the structure.

### Core Rule
The best comment is often a good name.

### Typical Refactorings
- **Extract Variable**
- **Extract Method**
- **Rename Method**
- **Introduce Assertion**

### Keep Comments For
- Explaining **why**, not **what**
- Documenting non-obvious constraints or tradeoffs
- Explaining an algorithm only after attempts to simplify it have failed

### Payoff
More intuitive code and less documentation drift.

---

## Duplicate Code

### Signs & Symptoms
- Two fragments look nearly identical
- Or they look different but perform the same job

### Why It Happens
Parallel work, copy-paste pressure, deadlines, and weak abstraction discovery.

### Why It Hurts
- Bugs are fixed in one place and missed in another
- Logic drifts apart subtly over time
- Maintenance effort multiplies

### Typical Refactorings
- **Extract Method**
- **Pull Up Field**
- **Pull Up Constructor Body**
- **Form Template Method**
- **Substitute Algorithm**
- **Extract Superclass**
- **Extract Class**
- **Consolidate Conditional Expression**
- **Consolidate Duplicate Conditional Fragments**

### Note
Do not create the wrong abstraction too early. Duplication is a smell, but a premature abstraction can be worse.

### Payoff
Shorter code, clearer structure, cheaper support.

---

## Lazy Class

### Signs & Symptoms
- A class does too little to justify its existence
- After refactoring, it became a tiny shell
- A class was created for anticipated future use that never arrived

### Typical Refactorings
- **Inline Class**
- **Collapse Hierarchy**

### Payoff
Reduced code size and less mental overhead.

### Note
A tiny class can still be justified if it expresses an important boundary or concept clearly.

---

## Data Class

### Signs & Symptoms
- A class is mostly fields plus getters/setters
- Behavior that should belong to the data lives in other classes

### Why It Happens
A class begins as a passive container and never grows into an actual object with behavior.

### Why It Hurts
- Business logic gets scattered into clients
- Violates “tell, don’t ask”
- Encourages anemic domain models

### Typical Refactorings
- **Encapsulate Field**
- **Encapsulate Collection**
- **Move Method**
- **Extract Method**
- **Remove Setting Method**
- **Hide Method**

### Payoff
Behavior moves next to the data it uses, improving cohesion and exposing duplicate client logic.

---

## Dead Code

### Signs & Symptoms
- Unused variable, field, method, parameter, branch, or class
- Obsolete behavior remains after changes in requirements

### Why It Hurts
Dead code creates noise, raises false questions, and makes the system appear more complex than it is.

### Typical Refactorings
- Delete unused code
- **Inline Class**
- **Collapse Hierarchy**
- **Remove Parameter**

### Payoff
Slimmer code, simpler maintenance, fewer distractions.

---

## Speculative Generality

### Signs & Symptoms
- Abstractions built “just in case”
- Unused hooks, parameters, fields, abstract classes, or extension points

### Why It Happens
Developers try to predict future needs and end up modeling possibilities instead of actual requirements.

### Why It Hurts
- Extra complexity today
- More code to understand and preserve
- Frequently the guessed abstraction turns out wrong

### Typical Refactorings
- **Collapse Hierarchy**
- **Inline Class**
- **Inline Method**
- **Remove Parameter**
- Delete unused fields

### When to Leave It Alone
Framework code may intentionally provide extension points for external users.

### Payoff
Leaner code and easier support.

---

# 5. Couplers

## Feature Envy

### Signs & Symptoms
- A method uses another object’s data more than its own
- The real knowledge and rules seem to belong somewhere else

### Why It Happens
Behavior was left behind when data moved, or the code never found the right home to begin with.

### Typical Refactorings
- **Move Method**
- **Extract Method**

### Heuristic
Things that change together should usually live together.

### When to Leave It Alone
Some patterns intentionally separate behavior from the data holder, such as Strategy or Visitor.

### Payoff
Better organization and less duplication of data-handling logic.

---

## Inappropriate Intimacy

### Signs & Symptoms
- One class reaches deeply into another’s internals
- Two classes know too much about each other’s fields or methods

### Why It Hurts
Tight coupling makes both classes harder to reuse, test, and evolve independently.

### Typical Refactorings
- **Move Method**
- **Move Field**
- **Extract Class**
- **Hide Delegate**
- **Change Bidirectional Association to Unidirectional**
- In some inheritance cases, **Replace Delegation with Inheritance** may apply

### Payoff
Cleaner boundaries and simpler maintenance.

---

## Message Chains

### Signs & Symptoms
- Calls like `a.getB().getC().getD()`
- Client code navigates object graphs directly

### Why It Hurts
The client becomes coupled to the internal structure of the collaboration. A structural change deep in the graph forces client changes.

### Typical Refactorings
- **Hide Delegate**
- **Extract Method**
- **Move Method**

### When to Leave It Alone
Over-hiding delegation can create a Middle Man. Balance matters.

### Payoff
Reduced dependency on object navigation paths and less bloated client code.

---

## Middle Man

### Signs & Symptoms
- A class mainly forwards calls to another class
- It adds little or no value of its own

### Why It Happens
Often appears after aggressively hiding delegates or after functionality migrated away over time.

### Typical Refactorings
- **Remove Middle Man**

### When to Leave It Alone
Some middle men are intentional and useful, especially in patterns like **Proxy** or **Decorator**, or when they protect an important boundary.

### Payoff
Less boilerplate and more direct code.

---

# 6. Other Smells

## Incomplete Library Class

### Signs & Symptoms
- A third-party or read-only library class almost fits your need but is missing crucial methods

### Why It Happens
You cannot modify the source library, yet your domain requires behavior that naturally belongs near that class.

### Typical Refactorings
- **Introduce Foreign Method** for a small missing helper
- **Introduce Local Extension** for broader missing behavior

### Payoff
Reduced duplication and a cleaner integration with the external library.

### Note
Extensions can increase maintenance cost if the library evolves frequently.

---

# Fast Smell-to-Refactoring Map

| Smell | Common Refactorings |
|---|---|
| Long Method | Extract Method, Replace Temp with Query, Replace Method with Method Object |
| Large Class | Extract Class, Extract Subclass, Extract Interface |
| Primitive Obsession | Replace Data Value with Object, Replace Type Code with Class/Subclasses/State |
| Long Parameter List | Introduce Parameter Object, Preserve Whole Object |
| Data Clumps | Extract Class, Introduce Parameter Object |
| Switch Statements | Replace Conditional with Polymorphism, State/Strategy |
| Temporary Field | Extract Class, Replace Method with Method Object |
| Refused Bequest | Replace Inheritance with Delegation, Extract Superclass |
| Alternative Classes | Rename Method, Extract Superclass, Move Method |
| Divergent Change | Extract Class |
| Shotgun Surgery | Move Method, Move Field, Inline Class |
| Parallel Inheritance Hierarchies | Move Method, Move Field |
| Comments | Extract Method, Rename Method, Extract Variable |
| Duplicate Code | Extract Method, Extract Class, Template Method |
| Lazy Class | Inline Class, Collapse Hierarchy |
| Data Class | Move Method, Encapsulate Field, Encapsulate Collection |
| Dead Code | Delete, Remove Parameter, Inline Class |
| Speculative Generality | Inline Method, Inline Class, Collapse Hierarchy |
| Feature Envy | Move Method |
| Inappropriate Intimacy | Move Method, Move Field, Hide Delegate |
| Message Chains | Hide Delegate, Move Method |
| Middle Man | Remove Middle Man |
| Incomplete Library Class | Introduce Foreign Method, Introduce Local Extension |

---

# Practical Review Checklist

When reviewing code, ask:

- Is this method doing more than one meaningful step?
- Is this class changing for more than one reason?
- Are related data and behavior separated?
- Is the same knowledge represented in multiple places?
- Would adding a new case require editing existing conditionals everywhere?
- Are we modeling domain concepts with raw primitives?
- Are objects talking through chains instead of through meaningful behavior?
- Is this abstraction solving today’s problem or a hypothetical future one?

---

# Refactoring Workflow

1. **Identify the smell**
2. **Protect behavior with tests**
3. **Choose the smallest safe refactoring**
4. **Refactor incrementally**
5. **Reassess** — one smell often reveals another underneath

Code smells are not a moral judgment. They are feedback. The goal is not perfect code; the goal is code that remains easy to change.

---

# Refactoring Techniques

This section adds the **refactoring techniques themselves**, so the guide is not only smell-oriented but also action-oriented. The techniques below are based on the PDF’s treatment catalog and grouped by purpose.

## How to Read This Section

For each technique, focus on:
- **Problem** — what situation triggers it
- **Solution** — the structural move you make
- **Use it when** — the practical signal
- **Watch out for** — common tradeoffs

---

## 1. Composing Methods

These techniques simplify long, tangled methods and improve readability.

### Extract Method
**Problem:** A code fragment can be grouped into a meaningful step.

**Solution:** Move that fragment into a new method and replace the original code with a call.

**Use it when:**
- a method has comments dividing sections
- a block expresses one business step
- the code is duplicated elsewhere

**Helps with:** Long Method, Duplicate Code, Comments, Feature Envy, Message Chains

---

### Inline Method
**Problem:** A method body is simpler and clearer than the method indirection itself.

**Solution:** Replace calls to the method with the method body, then delete the method.

**Use it when:**
- the method only delegates
- the name adds no meaning
- too many micro-methods obscure flow

**Helps with:** Speculative Generality, unnecessary indirection

---

### Extract Variable
**Problem:** An expression is hard to read.

**Solution:** Store part of the expression in a well-named variable.

**Use it when:**
- a conditional is dense
- an arithmetic or boolean expression is hard to parse
- naming the concept would improve readability

**Helps with:** Comments, readability problems inside long methods

**Watch out for:** extracting short-circuited boolean expressions can change evaluation behavior if the extracted pieces always execute.

---

### Inline Temp
**Problem:** A temporary variable only stores a simple expression once.

**Solution:** Replace the variable usage with the expression itself.

**Use it when:**
- the temp adds no clarity
- you are preparing for Replace Temp with Query or Extract Method

**Helps with:** unnecessary local noise

**Watch out for:** do not inline if the expression is expensive and reused.

---

### Replace Temp with Query
**Problem:** A local variable stores the result of an expression for later use.

**Solution:** Move the expression to a separate query method and call that method instead.

**Use it when:**
- a temp blocks method extraction
- the same expression is useful in multiple methods
- the logic deserves a name

**Helps with:** Long Method, Duplicate Code

**Watch out for:** the extracted method should not mutate visible state.

---

### Split Temporary Variable
**Problem:** One local variable stores multiple unrelated meanings over time.

**Solution:** Introduce separate variables, one per meaning.

**Use it when:**
- a variable is reassigned for different conceptual values
- names like `temp`, `value`, or `result` hide multiple roles

**Helps with:** readability, Extract Method preparation

---

### Remove Assignments to Parameters
**Problem:** A method reassigns one of its input parameters.

**Solution:** Copy the parameter into a new local variable and mutate the local instead.

**Use it when:**
- the parameter gets reused as working storage
- it becomes unclear what the original input still means

**Helps with:** Extract Method, readability, safer reasoning

---

### Replace Method with Method Object
**Problem:** A long method has so many tangled locals that Extract Method is difficult.

**Solution:** Turn the method into its own class; the former locals become fields, and the computation becomes methods on that class.

**Use it when:**
- extraction keeps failing because of intertwined state
- a single computation has grown into a mini-workflow

**Helps with:** Long Method, Temporary Field-like algorithm state

**Watch out for:** it adds a class, so use it when simpler method extraction is no longer viable.

---

### Substitute Algorithm
**Problem:** The existing algorithm is messy, obsolete, or inferior to a clearer one.

**Solution:** Replace the implementation with a new algorithm while preserving behavior.

**Use it when:**
- the old algorithm is harder to reason about than rewrite
- a library or simpler approach now exists
- requirements changed enough that patching the old version is wasteful

**Helps with:** Long Method, Duplicate Code

---

## 2. Moving Features Between Objects

These techniques improve cohesion by moving behavior and data closer to where they belong.

### Move Method
**Problem:** A method uses another class’s data more than its own.

**Solution:** Move the method to the class that owns most of the data it uses.

**Use it when:**
- you spot Feature Envy
- the method mostly navigates into another object
- moving it reduces coupling

**Helps with:** Feature Envy, Shotgun Surgery, Message Chains, Inappropriate Intimacy, Data Class

---

### Move Field
**Problem:** A field is used more by another class than by the one that owns it.

**Solution:** Move the field to the class where its behavior naturally lives.

**Use it when:**
- most methods touching the field live elsewhere
- the field is part of a responsibility being extracted

**Helps with:** Shotgun Surgery, Inappropriate Intimacy, Parallel Inheritance Hierarchies

---

### Extract Class
**Problem:** One class is doing the work of two or more concepts.

**Solution:** Create a new class and move the relevant fields and methods into it.

**Use it when:**
- a class has multiple reasons to change
- related fields and behavior form a coherent cluster
- a large class is mixing concerns

**Helps with:** Large Class, Divergent Change, Data Clumps, Primitive Obsession, Temporary Field, Inappropriate Intimacy

---

### Inline Class
**Problem:** A class has become too small to justify its existence.

**Solution:** Move its features into another class and delete it.

**Use it when:**
- a previous refactor left a shell class behind
- the class mostly delegates or stores almost nothing meaningful

**Helps with:** Lazy Class, Shotgun Surgery, Speculative Generality

---

### Hide Delegate
**Problem:** Client code calls through an object graph to reach another object.

**Solution:** Add a delegating method to the first object so the client stops depending on the chain.

**Use it when:**
- you see Message Chains
- clients know too much about internal relationships

**Helps with:** Message Chains, Inappropriate Intimacy

**Watch out for:** too much delegate hiding can create a Middle Man.

---

### Remove Middle Man
**Problem:** A class mostly forwards requests to another class.

**Solution:** Remove the delegating methods and let clients call the real target directly.

**Use it when:**
- a class contributes little beyond forwarding
- delegate methods are multiplying with no added value

**Helps with:** Middle Man

---

### Introduce Foreign Method
**Problem:** You need a utility/library method that you cannot add to the original class.

**Solution:** Add the helper method to a client class and pass the library object into it.

**Use it when:**
- you need one small missing method
- you are integrating with a read-only library

**Helps with:** Incomplete Library Class

---

### Introduce Local Extension
**Problem:** A third-party class needs several new methods, but you cannot change it.

**Solution:** Create a wrapper or subclass that adds the missing behavior.

**Use it when:**
- one foreign method is not enough
- the extension should be reusable across multiple clients

**Helps with:** Incomplete Library Class

---

## 3. Organizing Data

These techniques replace raw data handling with safer, richer models.

### Self Encapsulate Field
**Problem:** A class accesses its own fields directly, but you need more flexibility.

**Solution:** Route internal field access through getters/setters inside the same class.

**Use it when:**
- subclasses may need to redefine access
- you want validation, lazy init, or behavior at access time

**Helps with:** Duplicate Observed Data, some type-code refactorings

---

### Replace Data Value with Object
**Problem:** A primitive field now has behavior, rules, or related data.

**Solution:** Replace the primitive with a dedicated object.

**Use it when:**
- a string/number now carries business meaning
- logic around the primitive is duplicated
- the field is no longer “just data”

**Helps with:** Primitive Obsession, Duplicate Code

---

### Change Value to Reference
**Problem:** Many identical objects should really represent the same shared entity.

**Solution:** Replace repeated values with a shared reference object.

**Use it when:**
- you need one canonical entity instance
- updates should be visible everywhere that entity is used

**Typical examples:** user, product, order-like entities

---

### Change Reference to Value
**Problem:** A reference object is tiny and barely changes, so lifecycle management is unnecessary.

**Solution:** Make it an immutable value object.

**Use it when:**
- equality matters more than identity
- shared mutable state is adding friction

**Typical examples:** dates, money, ranges, colors, coordinates

---

### Replace Array with Object
**Problem:** An array stores different kinds of data in positional slots.

**Solution:** Replace it with an object whose named fields express meaning.

**Use it when:**
- `row[0]`, `row[1]`, `row[2]` require memorization
- array positions simulate a record structure

**Helps with:** Primitive Obsession

---

### Duplicate Observed Data
**Problem:** GUI/presentation classes also store domain data.

**Solution:** Split domain data into separate classes and synchronize them via observation.

**Use it when:**
- UI and domain behavior are entangled
- multiple views need the same underlying data

**Helps with:** Large Class, mixed UI/domain concerns

---

### Change Unidirectional Association to Bidirectional
**Problem:** Two classes both need access to each other, but only one side currently has the link.

**Solution:** Add the missing reverse association.

**Use it when:**
- reverse navigation is required frequently and computing it is expensive or awkward

**Watch out for:** bidirectional associations increase maintenance and coupling.

---

### Change Bidirectional Association to Unidirectional
**Problem:** One side of a two-way association no longer needs the other.

**Solution:** Remove the unused back-reference.

**Use it when:**
- the reverse association is dead weight
- reducing coupling improves reuse and clarity

**Helps with:** Inappropriate Intimacy

---

### Replace Magic Number with Symbolic Constant
**Problem:** A literal number has domain meaning but the code does not explain it.

**Solution:** Replace the literal with a named constant.

**Use it when:**
- the value is meaningful but not obvious
- the same literal appears repeatedly

**Watch out for:** not every number is magic; some are obvious enough in context.

---

### Encapsulate Field
**Problem:** A field is public.

**Solution:** Make it private and expose access through methods.

**Use it when:**
- you want control over reads/writes
- validation or invariant enforcement matters

**Helps with:** Data Class

---

### Encapsulate Collection
**Problem:** A class exposes a collection directly via getter/setter.

**Solution:** Return a read-only view and provide add/remove operations instead of wholesale external mutation.

**Use it when:**
- clients can mutate your collection behind your back
- the owner should control allowed operations

**Helps with:** Data Class

---

### Replace Type Code with Class
**Problem:** A primitive type code represents a concept but does not drive behavior.

**Solution:** Create a dedicated class for the type values.

**Use it when:**
- strings or ints encode roles, categories, or modes
- the type should gain validation or behavior

**Helps with:** Primitive Obsession

**Do not use it when:** the type code drives behavior branches; prefer subclasses or State/Strategy instead.

---

### Replace Type Code with Subclasses
**Problem:** A coded type directly affects behavior.

**Solution:** Replace type checks with subclasses that implement behavior polymorphically.

**Use it when:**
- object behavior differs by type
- inheritance is a natural fit for the variation

---

### Replace Type Code with State/Strategy
**Problem:** A coded type affects behavior, but subclassing is not appropriate.

**Solution:** Move the varying behavior into state/strategy objects.

**Use it when:**
- behavior changes dynamically
- subclass explosion would be awkward
- the host object should delegate variation instead of branching

---

### Replace Subclass with Fields
**Problem:** Subclasses differ only by simple constant-returning behavior.

**Solution:** Collapse the subclasses into fields on the parent.

**Use it when:**
- inheritance adds ceremony without meaningful behavioral richness

---

## 4. Big-Picture Selection Guide

### Use these first for readability
- Extract Method
- Extract Variable
- Replace Magic Number with Symbolic Constant
- Encapsulate Field

### Use these first for responsibility problems
- Extract Class
- Move Method
- Move Field
- Inline Class

### Use these first for primitive/domain modeling problems
- Replace Data Value with Object
- Introduce Parameter Object
- Replace Array with Object
- Replace Type Code with Class / Subclasses / State

### Use these first for coupling/navigation problems
- Hide Delegate
- Remove Middle Man
- Move Method
- Change Bidirectional Association to Unidirectional

### Use these first for algorithmic tangles
- Replace Temp with Query
- Split Temporary Variable
- Replace Method with Method Object
- Substitute Algorithm

---

## 5. Smell → Technique Decision Hints

- **Long Method** → start with **Extract Method**; if locals block you, try **Replace Temp with Query** or **Replace Method with Method Object**.
- **Large Class** → start with **Extract Class**; if the result becomes too fragmented, reassess and possibly **Inline Class** weak leftovers.
- **Primitive Obsession** → use **Replace Data Value with Object**, **Replace Array with Object**, or a type-code refactoring depending on whether behavior varies.
- **Long Parameter List** → use **Introduce Parameter Object** or **Preserve Whole Object**.
- **Data Clumps** → use **Extract Class** or **Introduce Parameter Object**.
- **Switch Statements** → prefer **Replace Type Code with Subclasses** or **Replace Type Code with State/Strategy**, then push toward polymorphism.
- **Temporary Field** → use **Extract Class** or **Replace Method with Method Object**.
- **Refused Bequest** → use **Replace Inheritance with Delegation** or **Extract Superclass** where appropriate.
- **Alternative Classes with Different Interfaces** → align with **Rename Method**, **Move Method**, **Add Parameter**, or **Extract Superclass**.
- **Divergent Change** → use **Extract Class**.
- **Shotgun Surgery** → use **Move Method**, **Move Field**, then **Inline Class** if old structures become empty.
- **Duplicate Code** → use **Extract Method**, **Pull Up Method**, **Extract Class**, or **Substitute Algorithm** depending on where the duplication lives.
- **Lazy Class** → use **Inline Class** or **Collapse Hierarchy**.
- **Data Class** → use **Encapsulate Field**, **Encapsulate Collection**, and **Move Method** to bring behavior home.
- **Dead Code** → delete it; also consider **Remove Parameter**, **Inline Class**, or **Collapse Hierarchy**.
- **Speculative Generality** → use **Inline Method**, **Inline Class**, **Collapse Hierarchy**, or delete unused fields.
- **Feature Envy** → use **Move Method**, sometimes after **Extract Method**.
- **Inappropriate Intimacy** → use **Move Method**, **Move Field**, **Hide Delegate**, or simplify associations.
- **Message Chains** → use **Hide Delegate** and sometimes **Move Method**.
- **Middle Man** → use **Remove Middle Man**.
- **Incomplete Library Class** → use **Introduce Foreign Method** or **Introduce Local Extension**.

---

## 6. Final Rule of Thumb

Refactoring techniques are not goals by themselves. They are **moves** you apply in response to a specific design pressure.

A good sequence is usually:
1. identify the smell,
2. protect behavior with tests,
3. pick the smallest refactoring that improves the design,
4. repeat until the code becomes easy to change.
