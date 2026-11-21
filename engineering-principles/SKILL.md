---
name: engineering-principles
description: "Professional software engineering principles (SOLID, DRY, KISS, YAGNI, modularity,composition). Use when writing, reviewing, refactoring, or designing code; when the user asks for clean code, coding standards, or professional engineering; or when friction appears (duplication, rigidity, surprise, shallow modules)."
---

# Engineering Principles

Load-bearing ideas beneath solid code. Reach for a principle when you feel **friction** — duplication, rigidity, surprise, train-wrecks, speculative abstractions — and let it name the fix.

**Speculative application inverts most principles into over-engineering.** Default to the simplest working design; justify each abstraction with a real axis of change or a proven third use.

## Agent workflow

### Writing or changing code

1. **Meet the need** with the boring design (KISS, YAGNI).
2. **Place knowledge once** — business rules, validation, invariants (DRY as knowledge, not line count).
3. **Draw boundaries** around decisions likely to change (information hiding); prefer **deep modules** (small surface, substantial interior).
4. **Depend on contracts**, not concrete types, only where substitution is real (second implementation, test double, new variant).
5. **Name and behave** so the least astonished reader is right (principle of least astonishment).

### Reviewing code

For each change, ask:

| Signal | Principle to apply |
|--------|-------------------|
| Same rule in two places that already diverged | DRY (knowledge) |
| Two similar blocks, different masters | AHA — wait for rule of three |
| Unrelated reasons to edit one module | Single responsibility |
| New `if` branch on stable code | Open–closed — extension point |
| Subtype breaks caller assumptions | Liskov |
| `not supported` stubs on interfaces | Interface segregation |
| Core imports DB/HTTP/clock directly | Dependency inversion |
| `a.b().c().d()` for behavior | Law of Demeter |
| Method reads and mutates opaquely | Command–query separation |
| Misuse is easy; docs are the only guard | Easy to use correctly, hard to use incorrectly |
| Many pass-through wrappers | Prefer depth over classitis |

End reviews with the [checklist](#pre-ship-checklist).

---

## SOLID

### Single responsibility

One **reason to change** per module — one actor, one axis of change.

- **Use when:** Unrelated edits keep hitting the same file; two teams own one class.
- **Watch out:** Anemic one-method types; “one responsibility” is not “one method.”

### Open–closed

Open for **extension**, closed for **modification** — add behavior via new code, not editing stable code.

- **Use when:** A module is stable and variations keep arriving; another `if` is the smell.
- **Watch out:** Extension points nobody uses; YAGNI applies to abstractions.

### Liskov substitution

Callers of a base type must work with **any** subtype; no strengthened preconditions or weakened postconditions.

- **Use when:** Designing inheritance or interface hierarchies — the test for real “is-a.”
- **Watch out:** Square/rectangle; subtype throws where parent did not → not a subtype.

### Interface segregation

Clients must not depend on methods they do not use — **role-sized** interfaces beat one fat interface.

- **Use when:** Implementers stub unused methods; one client’s API change recompiles everyone.
- **Watch out:** One interface per method is noise; segregate along real client roles.

### Dependency inversion

High-level **policy** and low-level **detail** both depend on **abstractions**; policy never imports detail.

- **Use when:** Business logic must run without database, queue, or clock; tests need fakes.
- **Watch out:** DIP ≠ dependency injection; DIP is **dependency direction**.

---

## Simplicity

### DRY (Don’t Repeat Yourself)

Every piece of **knowledge** has one authoritative representation.

- **Use when:** The same business rule lives in two places and has drifted.
- **Watch out:** Similar-looking code that changes for different reasons — deduplicating **couples** separate axes of change.

### KISS

Prefer the simplest design that meets the need; complexity is **spent**.

- **Use when:** Choosing boring-working vs clever-general — default boring.
- **Watch out:** Simple for the **reader**, not a cryptic one-liner for the writer.

### YAGNI

Do not build for presumed futures; build when the need is real; keep designs **addable** later.

- **Use when:** “While we’re in here, let’s also support…”
- **Watch out:** YAGNI is not permission to skip tests, errors, or clarity.

### AHA (Avoid Hasty Abstractions)

Prefer duplication over the **wrong** abstraction; abstract when the shape of commonality is **proven**.

- **Use when:** Unifying two paths that serve different masters.
- **Watch out:** Counterweight to DRY, not its replacement — shared knowledge still lives once.

### Rule of three

Tolerate duplication twice; on the **third** real use, refactor — three uses reveal the true abstraction shape.

- **Use when:** Deciding whether to extract helper, base type, or service.
- **Watch out:** Three trivial one-line repeats vs three copies of a **business rule** (the latter needed extracting yesterday).

---

## Object design

### Composition over inheritance

Assemble behavior from parts by reference; do not grow combinatorial class trees.

- **Use when:** Mixing, swapping, or configuring behavior at runtime.
- **Watch out:** Stable true “is-a” with LSP intact — inheritance can still be right.

### Program to an interface, not an implementation

Depend on the **contract**; stay ignorant of construction and concrete class.

- **Use when:** A seam may need substitution (persistence, transport, time).
- **Watch out:** Single forever-only implementation — add the seam when a second exists (including a fake).

### Command–query separation

**Queries** return data and change nothing observable; **commands** change state and return nothing meaningful.

- **Use when:** Designing APIs — composable reads, auditable writes.
- **Watch out:** Deliberate exceptions (pop, CAS, id-returning insert) — document them.

### Law of Demeter

Talk only to **immediate collaborators** — no behavioral train wrecks.

- **Use when:** Chains ripple changes through strangers; coupling through traversal.
- **Watch out:** Data (JSON, DTOs, fluent builders) is not behavioral Demeter territory.

### Principle of least astonishment

Behavior matches name, signature, and domain conventions; **surprise is a defect**.

- **Use when:** Naming, defaults, side effects — if users would be astonished, redesign.
- **Watch out:** Audience is the language and domain, not private taste.

---

## Modularity

### Information hiding

Boundaries around **decisions** that are hard or likely to change — not around flowchart steps.

- **Use when:** Drawing any module boundary — “what decision does this hide?”
- **Watch out:** Callers that know internals — nothing is hidden regardless of folder layout.

### High cohesion, low coupling

Related things together; unrelated things apart — optimize **both**, not either alone.

- **Use when:** Evaluating classes, services, or team boundaries.
- **Watch out:** One giant module has “perfect” coupling and terrible cohesion.

### Deep modules

**Simple interface**, substantial functionality behind it (Unix file I/O model).

- **Use when:** Judging decomposition — depth over shallow pass-through layers.
- **Watch out:** Classitis — interfaces multiply faster than complexity is hidden.

### Easy to use correctly, hard to use incorrectly

Right usage is the path of least resistance; misuse should not type-check or should fail fast.

- **Use when:** Every public API — types, ordering, defaults constrain mistakes.
- **Watch out:** Documentation cannot fix an interface that invites misuse.

### Design by contract

Preconditions, postconditions, invariants — executable obligations between caller and callee.

- **Use when:** Correctness-critical modules; contracts as docs, checks, and test oracles.
- **Watch out:** Stripping checks in production without an explicit decision removes the safety net.

### GRASP (responsibility assignment)

Thinking tool, not an architecture: **Information Expert** (behavior with the data), **Protected Variations** (instability behind interfaces), plus Creator, Controller, Low Coupling, High Cohesion, Polymorphism, Indirection, Pure Fabrication — use to assign responsibilities, not to manufacture layers.

---

## Composition

### Unix philosophy

Small tools that do one thing well, compose loosely, speak a simple universal interface.

- **Use when:** CLI, pipelines, services — sharp tools over integrated giants.
- **Watch out:** Composition moves complexity into **glue**; ten clever filters are one big program.

### Zen of Python (language-agnostic taste)

Explicit over implicit; simple over complex; **errors should never pass silently**; one obvious way where practical; practicality beats purity when aphorisms conflict.

---

## Pre-ship checklist

- [ ] One authoritative place for each business rule and invariant (DRY)
- [ ] No speculative features or extension points (YAGNI)
- [ ] Abstractions earned by proven repetition or a clear axis of change (AHA, rule of three)
- [ ] Module boundaries hide volatile decisions (information hiding, depth)
- [ ] Dependencies point inward to abstractions only where substitution is real (DIP, program to interface)
- [ ] Subtypes honor base contracts (LSP)
- [ ] APIs separate reads from writes unless deviation is intentional (CQS)
- [ ] Names and behavior match expectations (least astonishment)
- [ ] Misuse resisted by types and API shape, not comments alone (Meyers)
- [ ] Cohesion high, coupling low — no train-wreck behavior chains (Demeter)

## Further detail

Attributions, “builds on / leads to” links, and extended notes: [reference.md](reference.md).

Source catalog: [Engineering Principles](https://arafathusayn.github.io/engineering/).