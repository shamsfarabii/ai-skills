---
name: engineering-principles
description: "Apply software design principles (SOLID, DRY, KISS, YAGNI, AHA, modularity, composition) when writing, refactoring, reviewing, or designing code. Use when the user asks for a code review, refactor, clean or maintainable code, coding standards, or design/architecture advice, or when code shows a friction signal from the table below (duplicated business rules, growing if/switch chains, stubbed interface methods, core logic importing I/O, behavior call chains, misuse-prone APIs). Do not use for one-line fixes, pure syntax questions, or non-code tasks."
---

# Engineering Principles

**Core rule:** Default to the simplest design that works. Add an abstraction (interface, base class, extension point, helper layer) only if at least one is true:
- a second real implementation exists now (a test fake counts),
- the same logic has a third real use,
- a known, concrete axis of change exists (e.g. a second payment provider is scheduled).

Applying principles speculatively produces over-engineering. That is a defect, not extra quality.

## Workflow: writing or changing code

1. **Build the simplest working version.** Output: code that meets the stated need and nothing more. Add no unrequested options, hooks, configs or generality.
2. **Place each rule once.** Check whether each business rule, validation or invariant already exists in the codebase. If it does, call it; if not, define it in one place. Output: no copied rules.
3. **Scan the friction table.** Fix only signals that are actually present in your code.
4. **Check the reader's view.** Names, defaults and side effects should match what the signature implies.
5. **Run the pre-ship checklist.** In your reply, name any item you deliberately left unmet and say why.

## Workflow: reviewing code

1. **Read the change and its direct callers and callees.** Read enough to know how it is used, not the whole repo.
2. **Report each friction signal found** in this format:
   `location — signal — principle — concrete fix — severity (bug risk | maintainability)`
3. **Flag over-engineering with the same weight.** Examples: extension points with no second use, interfaces with one implementation and no fake, pass-through wrappers.
4. **End with the pre-ship checklist,** listing only the items that fail. If none fail, say so in one line.

## Friction table

| Signal | Principle | Fix | Don't over-apply |
|---|---|---|---|
| Same business rule in 2+ places, or copies have already diverged | DRY | Move it to one authoritative function or constant | Code that looks alike but changes for different reasons is not duplication |
| Two similar blocks serving different owners or use cases | AHA / rule of three | Keep them separate until a third real use shows the shared shape | Copied *business rules* get extracted now, not at three |
| One module edited for unrelated reasons, or owned by two teams | Single responsibility | Split along reasons to change | One responsibility ≠ one method; avoid anemic single-method types |
| Splitting or merging modules while judging only one of cohesion or coupling | High cohesion, low coupling | Group code that changes for the same reason; then check both cohesion and coupling | One giant module has "perfect" coupling and terrible cohesion; many tiny modules have the reverse |
| Unsure where a method belongs, or a method mostly reads another object's data | Information Expert (GRASP) | Move the behavior to the type that owns the data it uses | Don't invent Manager/Controller/Helper classes where a plain function would do |
| Each new variant adds an `if`/`switch` branch to stable code | Open–closed | Add an extension point (strategy, handler map) | Only when variants are actually arriving |
| Subtype throws, ignores, or narrows inputs where the parent didn't | Liskov substitution | Drop the inheritance; use composition or a separate type | — |
| Implementers stub methods as "not supported" | Interface segregation | Split the interface by client role | Not one interface per method |
| Business logic imports DB, HTTP, filesystem, clock, randomness, or a volatile third-party API directly | Dependency inversion / protected variations | Define an interface owned by the core; inject the adapter | This is about dependency *direction*, not DI frameworks |
| Class tree grows by combinations of features | Composition over inheritance | Compose behavior from parts | A stable, true is-a that honors Liskov can stay inheritance |
| `a.b().c().d()` used to trigger behavior | Law of Demeter | Ask the immediate collaborator for the result | Traversing data (DTOs, JSON, fluent builders) is fine |
| Function both returns data and mutates state | Command–query separation | Split into a query and a command | `pop`, compare-and-swap, id-returning insert are fine if documented |
| Misuse is prevented only by comments or docs | Easy to use correctly | Use types, required params and enums; fail fast on invalid input | — |
| Many thin layers that only forward calls | Deep modules | Merge layers: small interface, substantial interior | — |
| Callers rely on a module's internals | Information hiding | Put the volatile decision behind the module boundary | Boundaries follow decisions, not flowchart steps |
| A name, default or side effect would surprise a reader | Least astonishment | Rename it, or change the behavior to match the name | Judge by language and domain conventions, not personal taste |
| Errors caught and ignored, or behavior hidden in implicit magic | Explicit over implicit | Handle, propagate, or log with context; make behavior visible at the call site | — |
| Clever, dense code that is hard to read | KISS | Rewrite it plainly | Simple means simple for the reader |
| CLI tool, pipeline or service doing many jobs | Unix philosophy | Split into small tools that each do one job, with a simple shared interface (text, JSON) | Splitting moves complexity into glue code; ten clever filters are still one big program, so keep the glue small and explicit |
| "While we're here, let's also support…" | YAGNI | Don't build it; keep the design easy to extend later. Carrying unused code usually costs more than adding it when needed | Never a reason to skip tests, error handling or clarity |
| Correctness-critical preconditions or invariants | Design by contract | Keep the checks; remove them in production only by an explicit, stated decision | — |

## Edge cases

- **Design question, no code yet:** Apply the table to the proposed design. Output the recommended module boundaries and the decision each one hides.
- **Codebase conventions conflict with a principle:** Follow the codebase. Note the conflict and don't rewrite unrelated code.
- **Language without classes or interfaces (C, Go, functional, shell):** Apply the intent. Use function parameters, module boundaries and function types as the seams.
- **No tests, or the code can't be run:** Keep refactors minimal and behavior-preserving. State which changes are unverified.
- **User mandates a pattern or structure:** Do what they ask. State the trade-off once, without repeating it.
- **DRY and AHA conflict:** Shared *knowledge* (rules, invariants) gets extracted. Shared *shape* only stays duplicated.
- **Principles conflict in some other way:** Practicality beats purity. Choose what makes the code easier to change safely, and say which principle you set aside.
- **Prototype or throwaway script:** Apply only three things: one place per business rule, KISS, and no silently swallowed errors.
- **User asks where a principle comes from or why it exists:** Answer briefly from general knowledge (author, source, core idea), then tie it back to their code.

## Pre-ship checklist

- [ ] Each business rule and invariant lives in exactly one place
- [ ] No speculative features, options or extension points
- [ ] Every abstraction meets the core rule (second implementation, third use, or known axis of change)
- [ ] Behavior lives with the data it uses; modules are cohesive and loosely coupled
- [ ] Business logic doesn't import I/O, clock, randomness or volatile third-party APIs directly, where testing or substitution needs it
- [ ] Subtypes honor their parent's contract
- [ ] Reads and writes are separate, unless the exception is documented
- [ ] Names, defaults and side effects match what callers expect
- [ ] Misuse is blocked by types or API shape, not only by comments
- [ ] No ignored errors, and no behavior triggered through call chains