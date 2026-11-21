# Engineering principles — reference catalog

Condensed from [arafathusayn.github.io/engineering](https://arafathusayn.github.io/engineering/). Use when you need provenance, related principles, or deeper framing — not for every coding task.

## How to read entries

Each principle includes:

- **Essence** — the load-bearing idea
- **Use when** — friction signals
- **Watch out** — common over-application
- **Relations** — builds on / leads to / challenged by (where noted on the site)

---

## SOLID

### Single Responsibility Principle

Robert C. Martin · 2000 · acronym coined by Michael Feathers (~2004)

**Essence:** “There should never be more than one reason for a class to change.” Each module answers to one actor, one axis of change.

**Use when:** A class keeps changing for unrelated reasons, or two teams keep editing the same file.

**Watch out:** Over-splitting into anemic fragments; “one responsibility” means one reason to change, not one method.

**Relations:** Builds on high cohesion, low coupling. Challenged by CUPID over SOLID.

### Open–Closed Principle

Bertrand Meyer 1988 · restated by Robert C. Martin 2000

**Essence:** “Software entities should be open for extension, but closed for modification”: add behavior via new code, not by editing stable code.

**Use when:** A module has stabilized and variations keep arriving; expose an extension point instead of another if-branch.

**Watch out:** Speculative extension points nobody uses; YAGNI applies to abstractions too.

**Relations:** Challenged by CUPID over SOLID.

### Liskov Substitution Principle

Barbara Liskov 1987 · in SOLID via Martin 2000

**Essence:** Code using a base type must work, unknowingly, with any subtype; subtypes may not strengthen preconditions or weaken postconditions.

**Use when:** Designing any inheritance or interface hierarchy; it is the test for whether “is-a” is real.

**Watch out:** The classic square-extends-rectangle trap; if a subtype throws where the parent didn’t, it isn’t a subtype.

**Relations:** Challenged by CUPID over SOLID.

### Interface Segregation Principle

Robert C. Martin · 2000

**Essence:** “Clients should not be forced to depend upon interfaces that they do not use”: many small role interfaces beat one fat one.

**Use when:** Implementers stub out methods with “not supported”, or a change in one client’s methods recompiles all of them.

**Watch out:** An interface per method is noise; segregate along real client roles.

**Relations:** Challenged by CUPID over SOLID.

### Dependency Inversion Principle

Robert C. Martin · 2000

**Essence:** High-level policy and low-level detail both depend on abstractions; policy never imports detail directly.

**Use when:** Business logic needs testing without databases, queues, or clocks; invert so the core owns the interface.

**Watch out:** Don’t confuse with dependency injection (a technique); DIP is about which direction the source dependency points.

**Relations:** Builds on program to an interface, not an implementation. Leads to hexagonal architecture, composition root. Challenged by CUPID over SOLID.

---

## Simplicity

### DRY: Don’t Repeat Yourself

Hunt & Thomas · *The Pragmatic Programmer* · 1999

**Essence:** Every piece of knowledge should have a single, authoritative representation in the system.

**Use when:** The same business rule lives in two places and they have already drifted once.

**Watch out:** DRY is about knowledge, not lines; deduplicating code that merely looks alike couples things that change separately.

**Relations:** Leads to AHA.

### KISS: Keep It Simple

Folk engineering maxim · framed for software in Raymond’s *Art of Unix Programming* (2003)

**Essence:** Prefer the simplest design that meets the need; complexity is spent, not free.

**Use when:** Choosing between a working boring design and a clever general one: default to boring.

**Watch out:** Simple for the reader, not the writer; a terse one-liner can be the opposite of simple.

**Relations:** Builds on the Unix philosophy.

### YAGNI: You Aren’t Gonna Need It

Extreme Programming (Beck/Jeffries) · documented by Fowler

**Essence:** Don’t build capability for presumed future needs; build it when the need is real, and design so it can be added later.

**Use when:** Someone says “while we’re in here, let’s also support…”; cost of carry beats cost of later addition surprisingly often.

**Watch out:** YAGNI covers features and flexibility, not quality; it is not a license to skip tests or error handling.

**Relations:** Builds on “do the simplest thing that could possibly work.” Leads to speculative generality (as anti-pattern).

### AHA: Avoid Hasty Abstractions

Kent C. Dodds · after Sandi Metz’s “wrong abstraction”

**Essence:** Prefer duplication over the wrong abstraction; optimize for change first, and abstract only when the shape of the commonality is proven.

**Use when:** Tempted to unify two similar-looking code paths that serve different masters; wait for the third case.

**Watch out:** The counterweight to DRY, not its replacement; genuinely shared knowledge should still live once.

**Relations:** Builds on DRY, rule of three.

### Rule of Three

Refactoring folklore · recorded in Fowler’s *Refactoring*

**Essence:** Tolerate a duplication twice; on the third occurrence, refactor; three real uses reveal the true shape of an abstraction.

**Use when:** Deciding the moment to extract a helper, a base type, or a service.

**Watch out:** Three trivial repeats of one line may never need extracting; three copies of a business rule needed extracting yesterday.

**Relations:** Leads to AHA.

---

## Object design

### Composition over Inheritance

GoF · *Design Patterns* · 1994

**Essence:** “Favor object composition over class inheritance”: assemble behavior from parts held by reference rather than inherited from parents.

**Use when:** Behavior needs mixing, swapping, or configuring at runtime; inheritance would fork a combinatorial class tree.

**Watch out:** Inheritance is still right for true, stable is-a relationships with LSP intact.

**Relations:** Builds on GoF catalog. Leads to “the banana, the gorilla, and the jungle.”

### Program to an Interface, not an Implementation

GoF · *Design Patterns* · 1994

**Essence:** Depend on the contract a thing exposes, never on the concrete type behind it; callers stay ignorant of construction and class.

**Use when:** Any seam you may need to substitute: persistence, transport, formatting, time.

**Watch out:** Interfaces with a single forever-only implementation are ceremony; add the seam when a second implementation (including a fake) exists.

**Relations:** Leads to DIP, hexagonal architecture, dependency injection and composition root.

### Command–Query Separation

Bertrand Meyer · *Object-Oriented Software Construction*

**Essence:** Queries return a result and change nothing observable; commands change state and return nothing. Asking a question shouldn’t change the answer.

**Use when:** Designing any API; it makes reads freely composable and mutations easy to audit.

**Watch out:** Pragmatic exceptions exist (pop, compare-and-swap, id-returning inserts); deviate deliberately, not accidentally.

**Relations:** Leads to CQRS.

### Law of Demeter

Karl Lieberherr et al. · Northeastern · 1987

**Essence:** Principle of least knowledge: talk only to your immediate collaborators, no reaching through objects (`a.b().c().d()`).

**Use when:** Train-wreck call chains appear, or a change in one class ripples through strangers that merely traversed it.

**Watch out:** Doesn’t apply to data (JSON, DTOs, fluent builders); it’s about behavioral coupling, not dots.

**Relations:** Builds on high cohesion, low coupling.

### Principle of Least Astonishment

Long-standing usability/API maxim

**Essence:** A component should behave the way its name, signature, and context lead people to expect; surprise is a defect.

**Use when:** Naming things, choosing defaults, designing side effects: if users would be astonished, redesign.

**Watch out:** The audience defines astonishment: match the conventions of your language and domain, not your private taste.

---

## Modularity

### Information Hiding

David Parnas · CACM · 1972

**Essence:** Decompose systems around difficult or likely-to-change design decisions and hide each inside a module, not around the steps of a flowchart.

**Use when:** Drawing any module boundary: ask “what decision does this hide?”, not “what step does this perform?”.

**Watch out:** A module whose internals every caller knows hides nothing, whatever its folder structure says.

**Relations:** Leads to high cohesion, low coupling, deep modules.

### High Cohesion, Low Coupling

Stevens, Myers & Constantine · Structured Design · 1974

**Essence:** Keep related things together (cohesion) and unrelated things apart (coupling); the oldest quantitative lens on module quality.

**Use when:** Evaluating any decomposition: classes, services, teams; most architecture advice is this principle wearing a costume.

**Watch out:** Optimizing either alone is easy and useless; one giant module has perfect coupling and terrible cohesion.

**Relations:** Builds on information hiding. Leads to SRP, Law of Demeter, GRASP, DDD boundaries.

### Deep Modules

John Ousterhout · *A Philosophy of Software Design* · 2018

**Essence:** The best modules have simple interfaces hiding substantial functionality, like Unix file I/O: few calls masking enormous machinery.

**Use when:** Judging a design: prefer one deep module over many shallow pass-through classes and wrappers.

**Watch out:** “Classitis”: decomposition that multiplies interfaces faster than it hides complexity and makes systems shallower.

**Relations:** Builds on information hiding. Leads to “define errors out of existence.”

### Easy to Use Correctly, Hard to Use Incorrectly

Scott Meyers · IEEE Software · 2004

**Essence:** Interfaces should make the right usage the path of least resistance and misuse a compile error.

**Use when:** Every public API: constrain types, order, and defaults so mistakes don’t fit through the signature.

**Watch out:** Documentation is not a fix for an interface that invites misuse.

### Design by Contract

Bertrand Meyer · Eiffel · 1980s

**Essence:** Specify each routine’s preconditions, postconditions, and class invariants as executable obligations between caller and callee.

**Use when:** Correctness-critical modules; contracts double as documentation, runtime checks, and test oracles.

**Watch out:** Checks that vanish in production remove the safety net exactly where it matters; decide deliberately what stays on.

**Relations:** Leads to assertions, NASA’s Power of 10.

### GRASP: Nine Responsibility Principles

Craig Larman · *Applying UML and Patterns* · 1997

**Essence:** Information Expert, Creator, Controller, Low Coupling, High Cohesion, Polymorphism, Indirection, Pure Fabrication, Protected Variations: a checklist for assigning responsibilities to classes.

**Use when:** Deciding where behavior lives: give the job to whoever has the information (Expert), and wrap instability behind interfaces (Protected Variations).

**Watch out:** A thinking tool, not an architecture; don’t manufacture Controllers and Fabrications where a function would do.

**Relations:** Builds on high cohesion, low coupling.

---

## Composition

### The Unix Philosophy

Doug McIlroy · Bell Labs · documented 1978

**Essence:** “Write programs that do one thing and do it well. Write programs to work together. Write programs to handle text streams, because that is a universal interface.”

**Use when:** CLI tools, pipelines, microservices: anywhere small sharp tools composed loosely beat one integrated giant.

**Watch out:** Composition pushes complexity into the glue; a pipeline of ten clever filters is its own big program.

**Relations:** Leads to KISS, Zen of Python, middleware pipelines.

### The Zen of Python

Tim Peters · PEP 20 · 2004

**Essence:** Nineteen aphorisms: explicit is better than implicit, simple is better than complex, errors should never pass silently, there should be one obvious way to do it.

**Use when:** Language-agnostic taste calibration; “errors should never pass silently” alone justifies the read.

**Watch out:** Aphorisms conflict by design; “practicality beats purity” is in there to keep the rest honest.

**Relations:** Builds on the Unix philosophy.