# Engineering Standards — moss-piglet

> **Scope:** All implementation, debugging, refactoring, review, testing, and UI work in this repository, regardless of coding agent. For read-only tasks, inspect and explain without editing. Follow more specific applicable repository instructions when they do not contradict these baseline standards.

## Instruction precedence and judgment

- Follow the user's task scope and applicable repository instructions; never treat examples as verified APIs or paths.
- **Required:** correctness, strict task scope, evidence-backed changes, safe changes, verified design tokens, type safety, error handling, security, and truthful validation reports.
- **Preferred:** simple solutions, plain objects for simple string-keyed lookup, existing abstractions, and minimal changes. Deviate when technical requirements justify it.
- **Context-dependent:** extraction, memoization, virtualization, and backend changes. Require a concrete reason rather than applying them automatically.
- If repository reality conflicts with a rule, investigate the contradiction and explain the material decision. Do not silently rewrite unrelated legacy code.

---

## Core principle

Make the smallest readable change that fully solves the requested problem.
Preserve existing behavior outside that scope. Optimize for correctness,
maintainability, and appropriate resource use, not the fewest possible lines.

Follow applicable repository instructions and the user's requested scope. When
an existing pattern conflicts with these standards, inspect its purpose before
changing it. Explain consequential deviations; do not silently invent a new
architecture or expand the task into a repository-wide cleanup.

## 1. Work from evidence

### Inspect

- Read relevant repository instructions, the target implementation, its callers,
nearby examples, and affected tests. Search for existing solutions before
creating a component, hook, utility, type, token, or dependency.
- Inspect the current diff before editing. Preserve unrelated and user-authored
changes; never reset, overwrite, or reformat them to simplify the task.
- Check project configuration and scripts for the installed framework versions,
package manager, type checking, linting, and tests. Do not invent commands,
import paths, APIs, theme properties, or repository facts.

### Decide and implement

- Identify the requested behavior, existing behavior that must remain intact,
and the smallest affected surface. Trace a bug to its cause before patching it.
- For non-trivial work, briefly state the approach and important tradeoffs. For
a small fix, proceed without a separate planning document or approval loop.
- Reuse established architecture and patterns. Avoid unrelated refactoring,
dependency additions, file moves, formatting churn, and speculative helpers.
- Apply the relevant standards below; do not manufacture work to exercise every
section. For review-only or explanation-only requests, do not edit files.
- Resolve uncertainty from the code first. If a consequential requirement or
contract remains unknown, state the gap rather than inventing an answer.

### Stay within the task; do not hallucinate

- Treat the user-approved goal, acceptance criteria, and explicit constraints as
  the implementation boundary. Do not substitute a different goal, silently add
  features, or expand a fix into unrelated optimization or refactoring.
- Ground every change in the inspected code, actual types, available contracts,
  and observed behavior. Never invent an API, data field, hook behavior, file,
  dependency, performance result, or business requirement.
- Distinguish verified facts from assumptions and proposals. When a material
  unknown blocks a safe decision, explain it and request the one missing decision;
  do not fabricate details or implement speculative behavior.
- If a better approach changes the requested architecture or requires backend
  work, explain the reasoning and trade-offs **before** implementing that
  deviation. Do not silently switch approaches or edit outside authorized scope.
- Before finalizing, compare the diff against the original task and remove
  changes that do not directly support it or a necessary safety/correctness fix.

### Verify

- Inspect the final diff for unintended behavior, unrelated changes, and leftover
debugging code. Check affected callers when changing a shared contract.
- Run the relevant available checks using the repository's actual scripts.
Start with focused tests, then broaden type checking, linting, and build checks
according to the change's risk and the repository's requirements.
- Never claim a test, build, UI check, or performance measurement succeeded
unless it actually ran and its result was inspected. Report blocked or skipped
checks and distinguish known pre-existing failures from new ones.

## 2. Use the real design-token source

- Treat `STOCK_THEMES` in `app/util/stockThemes.tsx` as the project-design source
specified by these standards. Open it and representative consumers before
writing styles. If it has moved, locate the actual source instead of guessing.
- Use existing tokens for visual constants: color, typography, spacing, radii,
shadows, and other theme-controlled values. Do not hardcode spacing while
claiming a component is fully token-based.
- Reuse the token that matches the visual meaning, not merely a similar value.
Use theme-backed helpers only after verifying the repository's pattern.
- If a needed token is missing, add it to the theme source and associated types
first, preserving the shape of applicable theme variants. Then consume it.
- Resolve theme-dependent values from the active theme; do not freeze them in
module-level constants or introduce a parallel local palette.
- Keep non-styling values such as record counts and timestamps out of the theme.
Do not copy illustrative token paths from documentation into production code.

## 3. Keep TypeScript contracts explicit

- Do not introduce `any`, double assertions such as `as unknown as T`, or type
suppressions to hide a mismatch. Use `unknown` for untrusted values and narrow
them at the appropriate boundary using existing validation patterns.
- Use unions for finite domain states and discriminated unions when fields
depend on the state. Do not represent a closed set as an unrestricted string.
- Derive types from existing contracts with utilities such as `typeof`, `keyof`,
and `Pick` instead of maintaining parallel definitions.
- Type function parameters and exported function return values. Let obvious
local values use inference; do not add annotations that merely repeat it.
- Prefer narrowing or checked declarations to `as SomeType` and non-null `!`.
Where an assertion is unavoidable, verify the invariant and add a brief
explanation. Literal inference with `as const` is not an unsafe escape hatch.
- Put new feature types and interfaces in that feature's `types.ts`, or its
established equivalent. Reuse shared types from their owning module; do not
create a global type dumping ground or move unrelated legacy declarations.

## 4. Prefer typed objects for finite value lookup

Prefer a typed plain object when a finite string/number union maps to values. Keep the lookup in its owning feature's constants module beside the type, and reuse it across consumers. For general string-keyed dictionaries, use safe own-property access when keys are untrusted. Use `Map` when non-string keys, insertion/deletion semantics, or other concrete requirements justify it; do not ban `Map` categorically.

```ts
// types.ts - illustrative domain; reuse the actual feature's existing type.
export type Status = 'active' | 'pending' | 'failed';
```

```ts
// constants.ts
import type { Status } from './types';

export const STATUS_LABEL: Record<Status, string> = {
  active: 'Active',
  pending: 'Pending',
  failed: 'Failed',
};
```

For production text, follow the repository's existing copy/localization pattern.
For visual values, map to real typed token keys, then resolve the active theme.
Do not use `Record<string, ...>` when exhaustiveness over a known union matters.

Keep `if` or an exhaustive `switch` for genuine branching, validation, or
side effects. Do not turn control flow into a lookup table merely to satisfy a
style rule. Validate external values before treating them as a union member.

## 5. Model absence instead of hiding it

- Use optional chaining only for values that are genuinely optional or nullable.
Do not add chains or defaults simply to suppress an upstream contract error.
- Validate and narrow early. Distinguish expected absence from invalid data;
show an appropriate empty state for the former and an error for the latter.
- Do not silently turn malformed data into `''`, `[]`, `0`, or a successful empty
result. Use a fallback only when it represents intended product behavior.
- Guard each nullable level that matters. Checking `user` does not establish
that `user.profile` exists unless the validated type guarantees it.

## 6. Make failures explicit

- Catch errors where recovery, user feedback, cleanup, or useful context can be
provided. Never use an empty catch or a bare console message as handling.
- Treat caught values as `unknown`; narrow before reading properties.
- Preserve the original cause when wrapping an error, using the repository's
supported mechanism. Do not discard the stack or useful operation context.
- Use the existing error-reporting and notification paths. Give users a safe,
actionable message; keep diagnostic details in the appropriate logs. Redact
secrets and sensitive payloads; avoid duplicate logging and duplicate toasts.
- Route render failures through the existing error-boundary strategy. Handle
event-handler and arbitrary async failures through their actual caller or
query/mutation error path; do not assume a boundary automatically catches them.
- Distinguish loading, success, empty, and error states for reads. For mutations,
cover pending, success, and failure with safe retry behavior where applicable.
Never show a success message before the operation succeeds.

## 7. Keep components, hooks, and state simple

- Give each component a clear responsibility. Extract a hook when data access
or state coordination materially obscures the UI; do not split files solely
to meet an arbitrary line count or to wrap one straightforward expression.
- Reuse existing hooks and components. Extract an abstraction only when it
removes meaningful duplication or names a genuine responsibility, not merely
because two blocks look alike.
- Prefer a variant union over boolean props that allow contradictory states.
- Keep each state value under one owner. Derive values from existing state or
props instead of storing synchronized copies. Keep intentional editable drafts
explicit, including their initialization and reset behavior.
- Use the established server-state/cache mechanism (TanStack Query where already used), and keep local/client UI state in its existing owner (Jotai where appropriate). Do not mirror query data into another store without a concrete product requirement.
- Keep rendering pure and updates immutable. Do not use `useEffect` for values that can be derived during render. Use correct, complete hook dependencies;
clean up effects that subscribe, schedule, or start cancellable work. Prevent
stale responses from overwriting newer state where requests can overlap.
- Use stable, identity-bearing list keys. Do not use indices or generated random
keys for lists that can be reordered, filtered, inserted into, or edited.
- Add `useMemo` or `useCallback` only for a measured cost or a concrete reference-
identity requirement. Do not scatter memoization as a default optimization.

## 8. Choose React effects deliberately

For each proposed `useEffect` or `useLayoutEffect`, first explain what external
system or browser layout behavior requires synchronization. **Neither hook is
preferred by default**: prefer no effect when render logic or event handling
can solve the problem correctly.

- **No effect:** derive state from props/query data during render, perform user-
  initiated actions in event handlers, and use TanStack Query for server reads.
  Do not copy derived values into state or fetch through an effect merely to
  orchestrate data already managed by the query layer.
- **`useEffect`:** use for synchronization that does not have to finish before
  paint, such as subscriptions, timers, browser integrations, or non-visual
  external side effects. It is the default *when a real effect is needed*.
- **`useLayoutEffect`:** use only when layout measurement or a synchronous DOM
  adjustment must occur before the browser paints to prevent a visible jump or
  incorrect positioning (for example, measure-and-position a popover). Document
  the specific visual problem that an ordinary effect would cause. It blocks
  painting and can degrade responsiveness, so keep its synchronous work tiny.
- Avoid `useLayoutEffect` for data fetching, general state synchronization, or
  heavy calculations. Consider SSR/hydration behavior for code that may run
  outside the browser; do not use it as a workaround for poorly owned state.
- For either hook, verify complete dependencies, idempotent setup and cleanup,
  Strict Mode development replays, rapid mount/unmount, stale closures, and
  overlapping async operations. Never suppress dependency warnings to force a
  desired execution count.
- Where layout measurement is required, prefer existing refs and established
  observers/APIs; avoid repeated synchronous DOM reads and writes that trigger
  forced layout during interactions.

## 9. Keep rendering and network requests intentional

The objective is **necessary rendering and necessary requests**, not zero
re-renders or zero repeated calls. Optimize based on actual behavior, not
speculation or incidental Strict Mode development behavior.

- Trace which state/prop/query change causes each relevant render. Keep transient
  state local, subscribe to the smallest practical Jotai atom or query selection,
  and avoid broad state updates that invalidate unrelated views.
- Keep render functions pure. Do not set state or trigger network calls during
  render. Avoid effect-driven state loops, unstable dependency values, and
  unnecessary mirrored state that generates extra render passes.
- Use stable list keys. Preserve references only when required for a concrete
  consumer or measured hot path; use `React.memo`, `useMemo`, and `useCallback`
  selectively rather than wrapping everything by default.
- For network reads, use the existing TanStack Query cache and centralized query
  keys, with intentional enabled conditions, stale times, invalidation, and
  pagination. Check whether an existing cached result can be reused before
  introducing an additional endpoint call.
- Inspect the triggers for mount/refetch/focus/reconnect, repeated popover opens,
  parameter changes, and remounts. Prevent accidental duplicate effects,
  subscriptions, listeners, polling, or request waterfalls. Do not disable
  legitimate refreshes just to reduce request counts.
- Distinguish query-layer deduplication from mutation safety. For mutations,
  prevent unintended double submission and preserve retry/idempotency semantics;
  do not assume deduplication alone makes writes safe.
- For async work, handle cancellation/obsolete responses and races, especially
  when selection, filters, or pagination change quickly. Preserve expected
  loading, empty, and error UI states.
- Validate suspected performance problems with React Profiler, network traces,
  or existing project instrumentation when available. Record the trigger, input
  size, request count/render impact, and whether the behavior is expected.
  Never claim a measured improvement without before/after evidence.

## 10. Keep names and ownership clear

- Use concise domain vocabulary already present in the repository. Prefer
`isLoading`, `hasError`, `canSubmit`, and `handleThemeChange` to vague names,
unexplained abbreviations, or names that narrate the entire implementation.
- Separate types, constants, utilities, hooks, and components where that creates
a clear responsibility. Follow the feature-type convention in section 3;
do not create extra runtime files merely for organizational ceremony.
- Keep shared values, business rules, configuration, and validation in their
canonical owning modules. Import and reuse them instead of copying them.
- Keep dependency direction clear. Avoid circular imports and generic helper
modules that collect unrelated feature-specific behavior.

## 11. Treat accessibility as correctness

- Prefer existing accessible components and semantic elements. Do not replace
buttons or inputs with clickable containers that lack their behavior.
- Label inputs and icon-only controls. Preserve keyboard operation and visible
focus; maintain appropriate focus handling when temporary UI opens or closes.
- Do not convey status through color alone. Pair it with meaningful text or an
accessible icon, using the same design-token system as the rest of the UI.

## 12. Evaluate client cost and backend ownership

Before committing to an implementation, evaluate **frontend vs backend
ownership** for the behavior, calculation, aggregation, sorting/filtering, or
transformation being changed. If the backend is the more correct owner, clearly
**recommend that approach to the engineer before coding**; include why, the
contract impact, and any deployment/trade-off implications. Do not implement an
alternative frontend workaround without explaining the ownership mismatch.
Keep the assessment proportional: small UI-only display calculations do not
need architecture proposals.

- Keep interaction state, display formatting, and inexpensive UI-only derivation
in the frontend when they depend on local or unsaved user input.
- Prefer backend ownership for authoritative business rules, access-controlled
  filtering, cross-client consistency, large joins/aggregations, pagination, or
  expensive transformations over data already stored and processed there.
  Do not download an unbounded dataset to reconstruct backend-owned answers.
- Do not put large synchronous loops, heavyweight parsing, aggregation, sorting,
  or repeated full-dataset transforms on the main UI thread when the backend
  can perform the work correctly and efficiently. Where frontend processing is
  genuinely required (for example, unsaved local edits, offline work, or
  interactive client-only logic), bound its cost and consider chunking or a
  worker when measured responsiveness justifies the complexity.
- Consider whether a small response-shape change can remove repeated client-side
reconstruction. Balance that against payload size, network latency, caching,
deployment coordination, and other consumers of the contract.
- Do not pretend a proposed endpoint or response field already exists. Change
the contract only within the authorized scope. Otherwise use a correct,
bounded frontend implementation and state the backend improvement separately.
- Match data structures to realistic data sizes. Avoid repeated full scans,
unnecessary allocations, redundant requests, and large synchronous parsing or
serialization during frequent interactions.
- Keep render paths free of avoidable expensive work. Hoist only input-independent
values; use pagination, virtualization, or lazy loading when the actual scale
and existing architecture justify them, not by default.
- Compare before and after under comparable data and conditions for performance
work. Report the metric and input size. Without measurements, describe the
expected effect as a hypothesis, not a proven speedup.

Never sacrifice correctness, readable code, or maintainability for a speculative  
optimization or a shorter implementation.

## Security and external boundaries

- Treat API responses, user input, URL parameters, and persisted values as untrusted until validated using the repository's established schema/boundary patterns.
- Preserve authentication and authorization checks; frontend visibility is not authorization. Do not expose secrets, tokens, sensitive user data, or credentials in code, logs, or screenshots.
- Avoid introducing unsafe rendering, unvalidated redirects, or new network dependencies without inspecting existing security and timeout/retry conventions.
- Preserve mutation semantics and prevent unintended duplicate requests or actions where retries or user interaction can overlap.

## Completion checklist

Evaluate the applicable items before reporting completion; do not print this
entire checklist for every small task.

- [ ] The requested behavior is addressed without unrelated edits.
- [ ] Existing user changes, architecture, and unaffected behavior are preserved.
- [ ] Visual constants use verified tokens; new tokens cover applicable themes.
- [ ] Types, assertions, nullability, and value maps express the real contract.
- [ ] State, constants, business rules, and validation have clear owners.
- [ ] Effect choice (`useEffect`, `useLayoutEffect`, or neither) is justified;
      lifecycle, dependencies, cleanup, and layout impact are correct.
- [ ] Rendering, subscriptions, and API calls are intentional, with no avoidable
      duplicate requests, render loops, or race-induced stale updates.
- [ ] Frontend/backend ownership was evaluated and a better backend solution
      was raised before frontend implementation when relevant.
- [ ] Every changed behavior is grounded in code and within the requested scope;
      no invented contracts or unrelated work were introduced.
- [ ] Failure paths, accessibility, security boundaries, and realistic client-side costs are handled.
- [ ] Any backend dependency is real, authorized, or clearly marked as a proposal.
- [ ] Relevant checks ran, or limitations and remaining checks are disclosed.

## Final response and review findings

Keep the response proportional to the task. For implementation work, summarize
what changed, why any non-obvious decision was necessary, and what was verified.
Include unresolved risks or backend follow-ups only when they exist.

For reviews, lead with actionable findings ordered by severity. Identify the
file and line when available, the triggering condition, the impact, and the
smallest reasonable fix. Distinguish confirmed defects from hypotheses. When no
findings are supported, say so and mention verification limits rather than
inventing criticism.
