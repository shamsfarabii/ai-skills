---
name: moss-piglet-engineering-standards
description: Use for every implementation, debugging, refactoring, testing, UI, or code-review task in the moss-piglet repository. For read-only requests (review, explain, investigate), run steps 1 and 5 only and do not edit files. Does not apply to work outside this repository.
---

# moss-piglet Engineering Standards

**Core principle:** Make the smallest readable change that fully solves the requested task. Preserve all behavior outside that scope. Never invent APIs, fields, files, commands, tokens, or results.

**Precedence:** the user's task scope, then more specific repository instructions (if they don't contradict this file), then this file. Code samples here are illustrative, not verified APIs or paths.

## Workflow

### 1. Inspect (no edits yet)
- Read the target code, its callers, nearby examples, and affected tests. Search for an existing component, hook, utility, type, or token before creating one.
- Run `git status` and `git diff` and note any uncommitted user changes. Never reset, overwrite, or reformat them.
- Read `package.json` scripts and config to find the package manager and the real test, typecheck, lint, and build commands.
- For UI work, open `STOCK_THEMES` in `app/util/stockThemes.tsx` and two representative consumers.

**Output:** the files involved, the patterns to reuse, and the available check commands.

### 2. Decide
- State the requested behavior, the existing behavior that must not change, and the smallest affected surface. For a bug, trace it to its root cause before patching.
- **Ownership check:** if the change involves authoritative business rules, access-controlled filtering, large joins or aggregations, pagination, or expensive transforms of server data, the backend is likely the right owner. Recommend that before coding, covering the reason, the contract impact, and the deployment trade-offs. Skip this check for small UI-only display logic.
- **Stop and ask one specific question** if:
  - a requirement or contract is unknown and guessing would be consequential;
  - the better approach changes the requested architecture; or
  - the change needs backend work that wasn't authorized.

**Output:** for non-trivial work, a 2–4 sentence approach with trade-offs. For small fixes, proceed without asking for approval.

### 3. Implement
- Apply the Rules below that are relevant. Don't create work just to exercise every rule.
- Don't add unrelated refactors, dependencies, file moves, formatting churn, or speculative helpers.
- If existing code conflicts with a rule, inspect why before changing it. Do not silently rewrite unrelated legacy code.

### 4. Verify
- Re-read the full diff. Remove debug code, unrelated edits, and anything that doesn't serve the task or a necessary safety fix.
- If you changed a shared contract, check every caller.
- Run focused tests first. Then run typecheck, lint, and build as the risk warrants, using only the commands found in step 1.
- Record what ran and its result. Separate pre-existing failures from new ones; if unsure, check whether the failure occurs without your diff.

### 5. Report
- **Implementation:** what changed, why any non-obvious decision was made, which checks ran with their results, and which were skipped and why. Mention open risks or backend follow-ups only if they exist.
- **Review:** list findings ordered by severity. For each, give the file and line, the triggering condition, the impact, and the smallest fix. Label each as a confirmed defect or a hypothesis. If nothing is found, say so and state the limits of what you checked.
- Never claim a test, build, UI check, or performance gain that you didn't run and inspect.

## Rules

### Design tokens
- `STOCK_THEMES` is the source for all visual constants: color, typography, spacing, radii, and shadows. Don't hardcode these values or create a parallel local palette.
- Pick the token whose visual meaning matches, not just one with a similar value.
- If a token is missing, add it to the theme source and its types first, in every theme variant, then use it.
- Resolve theme values from the active theme at runtime, not from module-level constants. Non-styling values such as counts and timestamps stay out of the theme.

### TypeScript
- Never use `any`, `as unknown as T`, or `@ts-ignore`/`@ts-expect-error` to hide a mismatch.
- Use `unknown` for untrusted input and narrow it at the boundary with the repo's existing validation pattern.
- Avoid `as T` and non-null `!`. If one is unavoidable, add a comment stating the invariant that makes it safe. `as const` is fine.
- Model finite states as unions, and use discriminated unions when fields depend on the state.
- Derive types from existing ones with `typeof`, `keyof`, `Pick`, and similar utilities instead of writing parallel definitions.
- Type function parameters and exported return values. Let local values infer.
- Put new feature types in that feature's `types.ts` or its established equivalent. Import shared types from the module that owns them.

### Finite value lookups
- Map a finite union to values with a typed object in the feature's constants module:
  ```ts
  export const STATUS_LABEL: Record<Status, string> = { active: 'Active', pending: 'Pending', failed: 'Failed' };
  ```
- Use `Record<Union, …>` so every key must be covered. Don't use `Record<string, …>` for a known union.
- Validate external values before treating them as union members.
- Use `Map` only when a concrete need justifies it, such as non-string keys or insertion/deletion semantics.
- Keep `if`/`switch` for real branching, validation, or side effects.
- For user-facing text, follow the repo's existing copy or localization pattern.

### Absence and errors
- Use `?.` and defaults only for values that are genuinely optional. Never use them to silence a contract error.
- Never silently convert malformed data into `''`, `[]`, `0`, or a successful empty result. Show an empty state for expected absence and an error for invalid data.
- No empty catches, and no catch blocks that only log to the console.
  - Catch only where you can recover, clean up, give user feedback, or add context.
  - Narrow caught values from `unknown` before reading them.
  - Preserve the original cause when wrapping an error.
- Report errors through the existing error-reporting and notification paths.
  - Show users a safe, actionable message.
  - Redact secrets.
  - Avoid duplicate logs and duplicate toasts.
- Route render errors through the existing error boundary. Handle event-handler and async errors at their caller or in the query/mutation error path.
- Reads need distinct loading, success, empty, and error states. Mutations need pending, success, and failure states, and must never show success before the operation succeeds.

### React state, effects, and rendering
- Each piece of state has one owner.
  - Server data lives in TanStack Query with centralized query keys.
  - Client UI state stays with its existing owner (Jotai where already used).
  - Don't mirror query data into another store, or copy derivable values into state.
- Before any `useEffect` or `useLayoutEffect`, write one sentence naming the external system it synchronizes with.
  - Use no effect when render-time derivation, an event handler, or TanStack Query can do the job.
  - Use `useEffect` for subscriptions, timers, and browser integrations.
  - Use `useLayoutEffect` only for measurements or DOM adjustments that must happen before paint to prevent a visible jump. Name that jump in a comment, keep the work tiny, and consider SSR.
- For any effect:
  - Declare complete dependencies, and never suppress the lint warning.
  - Make setup and cleanup idempotent so it survives Strict Mode replay and rapid mount/unmount.
  - Prevent stale or overlapping responses from overwriting newer state.
- Keep render pure: no setState and no network calls during render.
- Use stable, identity-based list keys, never indices or random values.
- Add `useMemo`, `useCallback`, or `React.memo` only for a measured cost or a concrete reference-identity need.
- Prefer variant unions over boolean props that can contradict each other.
- Extract a hook or component only when it names a real responsibility or removes meaningful duplication.

### Network
- Before adding a request, check whether existing cached query data already answers it.
- Set `enabled`, `staleTime`, invalidation, and pagination deliberately.
- Check what triggers refetches (mount, focus, reconnect, remount, repeated opens) and remove accidental duplicates. Don't disable legitimate refreshes.
- Query deduplication does not make writes safe. Prevent double-submitting mutations and preserve their retry and idempotency semantics.

### Accessibility
- Use semantic elements and the existing accessible components. Never make a `div` act as a button.
- Label every input and every icon-only control. Keep keyboard operation and visible focus working, and manage focus when temporary UI opens or closes.
- Never convey status through color alone; pair it with text or an accessible icon.

### Security
- Treat API responses, user input, URL parameters, and persisted values as untrusted until validated.
- Keep existing auth checks in place; hiding something in the frontend is not authorization.
- Never put secrets, tokens, or sensitive user data in code, logs, or screenshots.
- No unsafe HTML rendering, no unvalidated redirects, and no new network dependencies without first checking the existing security and timeout conventions.

### Naming and ownership
- Use the domain vocabulary already in the repo, with names like `isLoading`, `hasError`, `canSubmit`, and `handleX`.
- Keep business rules, config, and validation in their owning module and import them rather than copying.
- Avoid circular imports and catch-all helper files.

### Performance claims
- Keep expensive work out of render paths and the main thread when the backend can do it correctly. If frontend processing is required, bound its cost.
- Add virtualization, pagination, or workers only when the real data size justifies them.
- Claim an improvement only with before/after measurements (Profiler, network trace) that report the metric and input size. Without measurements, call it a hypothesis.

## Edge cases
- **No script for a check:** don't invent one. Report the check as unavailable.
- **`STOCK_THEMES` not at its listed path:** search the repo for `STOCK_THEMES`. If it isn't found, stop and ask where design tokens live.
- **Checks fail before your change:** report them as pre-existing, and don't fix them unless they're in scope.
- **Uncommitted user changes in files you touch:** edit around them and mention them in the report.
- **The feature has no `types.ts` or equivalent:** create one only for the new types you add. Don't move legacy declarations into it.
- **Backend is the right owner but not authorized:** implement a correct, bounded frontend version, and list the backend change separately as a proposal. Never treat a proposed endpoint or field as if it exists.