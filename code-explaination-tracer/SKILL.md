---
name: code-explanation-tracer
description: Explain what existing code does and why, in plain language anyone can follow. Use when the user shares or points to code (a snippet, function, component, hook, class, SQL query, regex, shell script, config, algorithm, or whole file, in any language) and asks what it does, how it works, how data flows through it, why it is written this way, or why it produces a certain result or bug. Also use when they want to understand code before reviewing or changing it. Works for any experience level, including non-programmers. Do not use for writing new code or refactoring unless the user first wants the existing code explained.
---

# Code Explanation Tracer

Goal: the reader understands **why the code exists** and can **predict its output for a new input**. Explain purpose before syntax. Show real values before abstract descriptions.

## Steps

Work through these in order. Each step's output becomes one section of the answer, using the heading shown.

### 1. Read and ground (internal, no heading)
- Read the code the user supplied. If a repository or files are available, open only the callers, types, and helpers that change the behavior.
- List for yourself what you **saw** and what you **did not see**, such as unseen helpers, imports, or config.
- **Output:** a short internal note of the code's inputs, outputs, side effects (network, database, file, DOM, or state changes), and any unknowns.

### 2. Set the reader level (internal, no heading)
- If the user stated their level, or their wording makes it obvious, match it.
- Otherwise, default to **plain**: assume the reader is smart but may not know this language or framework.
- **Output:** the chosen level. Plain level applies the rules under "Plain-language rules" fully. Expert level may skip basic definitions.

### 3. "What problem does this solve?"
- Write 1–2 sentences on what the code does, in everyday words with no jargon.
- **Without it:** describe one concrete failure or missing capability.
- **With it:** describe what changes for the user or the system.
- If the purpose is not stated in the code, comments, tests, or the user's message, start that line with "Likely:". Never invent bug reports, business requirements, or past design decisions.
- Add a one-line flow like `click -> handler -> filter -> setState -> re-render`, using only steps that actually appear in the source.

### 4. "Step by step with an example"
- Build the smallest realistic mock input, usually 2–4 items, with names and values that make changes easy to spot. Label it "example data".
- Trace it through the **actual** statements in execution order. After each meaningful step, show the current value of the variables that changed.
  - For loops, use a table with one row per iteration and columns for the variables that change.
  - For object or state updates, show the value before and after.
  - For async code (callbacks, promises, `await`, events, effects), show a numbered timeline that separates when code is **declared**, when it **runs**, and when it is **cleaned up**.
- State the exact final value or visible effect, and explain why it follows from the trace.
- If another branch would change the result (an `if`, an early `return`, a `catch`), trace it briefly with a second input.

### 5. "Why it's written this way" and "What could go wrong"
- For each important construct (for example a `Map`, a memo, a debounce, a transaction, a retry, an index, or a regex group), say what it does **here**, not its textbook definition.
- Show one simpler or naive alternative and what concretely breaks or gets worse with it. Then state the cost of the current approach.
- Do not claim performance gains without a measurement or a complexity argument.
- If the code is flawed or more complicated than it needs to be, say so plainly. Do not invent a justification.
- List 2–4 edge cases that actually apply, such as empty input, missing or null values, duplicates, case sensitivity, errors, race conditions, stale closures, security, or cleanup.
  - For each one, give a tiny input, what the code does, and whether that is handled or is a gap worth testing.

### 6. "In one sentence"
- Give an accurate one-sentence mental model.
- Then give one fresh input and ask the reader to predict the result, with the answer shown below it.

## Plain-language rules

- Define every technical term the first time it appears, in a few words in parentheses. Example: "callback (a function handed over to be run later)".
- Keep to one idea per sentence. Prefer short words.
- Use an everyday analogy only when it clarifies something, then immediately map it back to the code. Example: "Like a sticky note on the fridge: `cache` remembers the last answer so we don't recompute it."
- Show values, not descriptions. Write "`total` goes from `0` to `12`" rather than "the total is updated".
- Quote short pieces of the source when discussing them. Do not paste the whole file back.

## Matching depth to the code

- **Under ~20 lines:** merge sections 3–5 into a few short paragraphs, keeping the trace. Skip the trade-off discussion if there is no meaningful alternative.
- **Whole file or more than ~150 lines:** start with a map of its main parts (one line each). Trace the single most important path in full. Summarize the other parts briefly, and offer to trace any of them next.
- **The user asks for every line:** go through each meaningful statement or block in source order, giving three things:
  - **What:** what the statement does.
  - **Why:** why the code needs it.
  - **Effect:** how the values change, using the example data.

  Skip trivial boilerplate unless the user asks about it.
- **The user asks about a bug:** reconstruct the failing path with an input that triggers it. Then say how to reproduce it and how to verify a fix. Suggest a fix only after that, and keep it separate from the explanation.

## Edge cases in the request itself

- **No code is visible:** ask the user to paste it or name the file. Do not guess.
- **A helper or import is not shown:** write "The behavior of `helperName` can't be confirmed from this code." Then describe only what the call site proves, such as its arguments and how its result is used.
- **The code looks truncated or partial:** explain what is present and note where it cuts off.
- **The library or framework version is unclear and it matters:** state the assumption you are making, for example "Assuming React 18 behavior".
- **The code was not executed:** you traced it by reading, so say "traced by reading" if anyone could think it was run. Never claim to have run code you did not run.
- **The name, comment, and implementation disagree:** point out the contradiction. Explain what the code actually does.
- **The code is minified, obfuscated, or pseudocode:** say so, and rename variables in your explanation for readability while showing the mapping.
- **Key behavior depends on missing context:** ask **one** targeted question. If a useful explanation is possible without it, give the explanation and state the uncertainty instead of asking.

## Correctness rules (always apply)

- Preserve the code's exact semantics in the trace: evaluation order, mutation versus copy, type coercion, reference equality, integer versus float behavior, return order, and exceptions.
- Keep example data consistent across every step.
- Keep proven behavior separate from intended or likely behavior.
- Do not rewrite or refactor unless asked. Put optional improvements in a clearly labeled final note.

## Before sending, check

- Can a reader who doesn't know this language follow section 3?
- Does every value in the trace follow from the code as written?
- Is every guess labeled "Likely:" or stated as an assumption?
- Does the prediction exercise have a correct answer?

## Reference example

```ts
function getReadyEmails(rows: { email: string; status: string }[]) {
  return rows
    .filter((row) => row.status === "ready")
    .map((row) => row.email);
}
```

- **Problem:** the caller needs only the email addresses of rows that are ready, not the whole row objects.
- **Example data:** `[{email:"a@x.com",status:"ready"}, {email:"b@x.com",status:"invalid"}, {email:"c@x.com",status:"ready"}]`
- **Step 1:** `filter` (keeps items that pass a test) keeps rows a and c → `[{a…}, {c…}]`.
- **Step 2:** `map` (turns each item into something else) extracts the email from each → `["a@x.com", "c@x.com"]`.
- **Why two steps:** `filter` decides *which* items stay and `map` decides *what shape* to return. A single `reduce` could do both, but it would be harder to read for a case this simple.
- **Edge cases:** the comparison is case-sensitive, so `"READY"` is excluded. An empty array returns `[]`.
- **In one sentence:** keep the ready rows, then pull out just their emails.
- **Predict:** for `[{email:"d@x.com",status:"Ready"}]`, the result is `[]`, because `"Ready"` does not exactly match `"ready"`.