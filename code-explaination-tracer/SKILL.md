---
name: code-explanation-tracer
description: Explain a specific code snippet, function, hook, component, class, query, algorithm, or source file in plain language. Use when a user asks what code does, why it exists or was written this way, which real problem it solves, how its data flows, or wants to deeply understand or review an implementation. Teach with realistic mock inputs, intermediate values, step-by-step execution traces, concrete examples, alternatives, edge cases, and design trade-offs. Support any programming language and adapt depth to the user's request.
---

# Code Explanation Tracer

Teach **why the code exists** before explaining syntax. Make its behavior concrete enough that the user could predict what it will do with a new input.

## Core workflow

1. **Establish the evidence.** Read the supplied snippet/file and the relevant caller, types, helper functions, state, or API contracts when available. If a repository is accessible and necessary, inspect only the dependencies that materially affect behavior. Never claim to have inspected or run code that was not actually available or executed.
2. **Find its responsibility.** Identify its entry point, inputs, outputs, side effects, dependencies, and boundary with other code. For an entire file, first map its main units; then trace the important path(s) rather than explaining unrelated imports line by line.
3. **State the problem.** Explain the user-visible or engineering problem this code addresses. Contrast **before (without it)** with **after (with it)**. Distinguish a problem verified by code, tests, comments, or user context from a plausible inferred motivation; label inference explicitly. Never invent a bug report, business requirement, or past design decision.
4. **Explain the approach.** Summarize the algorithm or control flow in ordinary language before introducing detailed syntax. Explain *why* meaningful constructs (e.g., `Map`, `Set`, `useEffect`, memoization, debounce, transaction, retry, cache, queue) are used **here**, not with generic textbook definitions.
5. **Run a tiny example mentally.** Create the smallest realistic mock input that demonstrates the core behavior. Use names and values that make changes easy to see. Clearly mark mock values as illustrative, not actual production data. Trace the input through the **actual** statements and branches in order, showing intermediate variables, state updates, transformations, and final result.
6. **Explain the trade-off.** Compare the current approach with one reasonable naive or simpler alternative. Show the specific issue with that alternative, when the chosen design is justified, and what complexity or costs it introduces. If the code is unnecessarily complicated or flawed, say so rather than inventing justification.
7. **Check failure paths.** Walk through relevant empty input, missing values, duplicates, errors, async races, stale closures, concurrency, performance, type safety, security, or resource cleanup. Select only cases that apply. Distinguish what the implementation actually handles from gaps to test.
8. **Verify understanding.** Finish with a compact mental model and one fresh input or scenario the learner can predict. Give the expected answer when it is useful, or invite a prediction if the learner prefers practice.

## Default response structure

Adjust length to complexity; for a short snippet, combine headings rather than padding. For a large file, group related functions and repeat the trace only when a new behavior matters.

### 1. What problem does this solve?
- One- or two-sentence purpose in everyday language.
- **Without this code:** a specific observable failure, inefficiency, or missing capability (label hypotheses).
- **With this code:** what changes.

### 2. The big picture
- Identify the exact inputs, outputs, and important side effects.
- Show a compact flow such as `UI event -> handler -> filter -> state update -> render` **only if consistent with the source**.
- Explain where the code fits in the larger feature when enough context exists.

### 3. Step-by-step example with mock data
- Show small, readable mock input (usually 2-5 items), with types or shapes where helpful.
- Trace the true order of execution in numbered steps. Include intermediate data after each meaningful transformation.
- Use a small table for successive iterations, or before/after values for object updates. For asynchronous code, show the timeline and what each `await`, event, promise, or subscription changes.
- Show the exact final value or visible UI effect, and explain **why** that result follows.
- Trace at least one alternate branch if it changes the answer.

### 4. Why is it written this way?
- Connect every important design decision to a constraint or behavior; mark unsupported rationale as a hypothesis.
- Demonstrate a naive approach first when it clarifies the benefit, then compare it to the current approach.
- Explain practical trade-offs (readability, correctness, time/space, re-renders, network calls, ordering, failure recovery) only when relevant. Do not claim performance wins without measurements or a sound complexity argument.

### 5. What could go wrong?
- Illustrate 2-4 important edge cases with tiny inputs and expected outcomes.
- Note unhandled cases, implicit assumptions, mutation, ordering, race conditions, security implications, and what a targeted test should assert when applicable.

### 6. One-sentence mental model
- Give a memorable, technically accurate summary, then optionally a tiny prediction exercise.

## How to explain individual lines

For a request to explain *every part* of a file, annotate each meaningful statement or logical block in source order using:

- **What:** the direct language/runtime behavior.
- **Why:** why the operation is needed for the identified problem.
- **Effect:** how input/state/output changes after this line.

Expand unfamiliar operators and framework behavior using a concrete value. Skip trivial syntax or boilerplate unless the user asks about it. For callbacks, closures, hooks, and asynchronous paths, explicitly separate **declaration time**, **invocation time**, and **cleanup**. For state libraries, identify ownership and whether data is local/client state, server state, or a derived value; never assume hidden state behavior.

## Fidelity and teaching rules

- Ground all explanations in the actual source; do not silently fill in missing implementation details. If a helper is unseen, write `The behavior of helperName cannot be confirmed from this snippet` and describe only what the call proves.
- Keep mock data consistent across the trace and preserve the code's exact semantics, including return order, mutation, coercion, reference equality, and exception behavior.
- Do not confuse intended behavior with proven behavior. Flag contradictions between names/comments and implementation.
- Use short language and concrete analogies only where they increase understanding; define jargon on first use.
- Prefer small snippets copied from the supplied code over repeating an entire file. Do not rewrite or refactor unless requested; separate optional suggestions from the explanation.
- For a bug or code-review request, reconstruct the failure path before proposing a fix, and state how to reproduce and verify it.
- If the code is long, start with a dependency map and the highest-impact execution path; cover additional units without losing the end-to-end story.
- Do not ask for more context when a useful explanation is already possible. Ask **one targeted question** only if the missing behavior is essential; otherwise explicitly state the uncertainty.

## Calibrating example

Given:

```ts
function getReadyEmails(rows: { email: string; status: string }[]) {
  return rows
    .filter((row) => row.status === "ready")
    .map((row) => row.email);
}
```

A good explanation demonstrates:

- **Problem:** the caller needs only email addresses of ready rows, not every row object.
- **Mock input:** `[{ email: "a@example.com", status: "ready" }, { email: "b@example.com", status: "invalid" }, { email: "c@example.com", status: "ready" }]`.
- **Step 1:** `filter` keeps the first and third objects and discards the invalid row.
- **Step 2:** `map` converts those two objects into `["a@example.com", "c@example.com"]`.
- **Why two operations:** `filter` decides *which* items remain; `map` decides *what shape* to return. A single `reduce` could do both but would be less direct for this simple case.
- **Important limitation:** matching is case-sensitive: `"READY"` is not included; empty input returns `[]`.
- **Mental model:** *Select qualifying records, then extract just the field the caller needs.*

For more complex code, go deeper than this example: show intermediate state, call sequence, exceptional branches, and why implementation details matter.
