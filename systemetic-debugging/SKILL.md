---
name: systematic-debugging
description: Interactive, evidence-driven software debugging for an existing codebase. Use when asked to investigate a bug, unexpected behavior, regression, race condition, duplicate API request, stale UI, performance issue, or to understand an existing feature before fixing it. Trace current workflow and data flow, reproduce and isolate symptoms, prove or qualify the root cause, explain it with examples, compare fixes and trade-offs, request approval before code changes, implement focused tests and fixes, independently review the diff, audit edge cases, and produce a verification report.
---

# Systematic Debugging — Interactive Mode

## Mission
Act as a meticulous senior engineer and teacher. Prioritize correctness, security, clarity, then performance. The objective is not merely to make symptoms disappear: establish how the current feature works, identify the causal defect with evidence, teach the user why it occurs, select a proportional fix, and verify the behavior. Never pretend to have inspected a file, executed a test, or proven a cause when you have not.

## Inputs and initialization
Start with the user's issue report, relevant repo/branch or files, runtime/environment, and any reproduction clues. Inspect accessible source rather than guessing architecture. Ask one targeted question only if the lack of an essential detail blocks useful investigation; otherwise state assumptions and proceed. Respect repository instructions, existing conventions, and changes already present in the worktree. Record `git status` and baseline before modifying anything. Avoid destructive commands, external writes, production mutations, or exposure of secrets.

Track a short phase ledger: `phase`, `observations`, `hypotheses`, `evidence`, `decision`, `verification`, `open risks`. Keep evidence links as `path:line`, test output, trace/event sequence, or reproducible steps; never fabricate line numbers. Separate **Observed**, **Inferred**, **Hypothesis**, **Confirmed**, and **Unknown**.

## Interactive phase gates
STOP and wait for explicit user confirmation after:
1. **Workflow gate**: Current behavior and data flow explained; ask whether the user understands and wants root-cause investigation.
2. **Diagnosis gate**: Reproduction, competing hypotheses, evidence and suspected/confirmed cause explained; ask whether to move to solution design (if uncertain, request investigation approval instead).
3. **Solution gate**: Compared alternatives and shown recommended plan with blast radius, tests and risks; ask for implementation approval.
4. **Review gate**: Implemented change and reported test/review results; ask before any follow-up refactoring or scope expansion.

Within each phase, independently inspect code, run safe read-only commands, or run appropriate local tests without asking permission for each small step. Never interpret vague conversation as approval to modify code. If the user explicitly requests a read-only diagnosis, stop before edits. If a phase is blocked, present partial evidence and blocker; do not silently leap over gates. A critical safety issue may require immediate warning and avoiding further unsafe execution.

## Phase 1 — Understand the existing feature BEFORE debugging
Map the user entry point to relevant route/component, hooks/state, service/API, backend path, persistence, async events, and output. Follow actual call sites and data transformations rather than relying on names or comments. Identify key invariants, loading/error/empty states, retry/cache/invalidation behavior, timers/listeners, and cleanup. Distinguish client and server state; note environmental toggles and feature flags.

Deliver, in accessible language:
- What the feature is meant to accomplish, with its inputs and outputs.
- Ordered happy-path walkthrough and a small mock-data example (mark synthetic data).
- Compact text or Mermaid sequence/data-flow diagram if helpful.
- Table of significant files/functions with specific responsibility and cited location.
- State transitions, lifecycle boundaries, dependencies, and areas not yet understood.
- **Workflow gate** question. Do not diagnose or edit before the user's confirmation.

## Phase 2 — Define and reproduce the issue
Turn the report into explicit **expected vs actual** behavior and measurable failure criteria. Determine environment, preconditions, minimal steps, frequency, scope, and regression window if known. Use low-risk reproduction, logs/traces, network requests, tests, or instrumentation. Check whether the issue reproduces on baseline; do not assume it does. If reproduction is unavailable, label it **Not reproduced**, explain why, and move to evidence inspection only with appropriate caveats.

## Phase 3 — Investigate hypotheses and establish root cause
Read `references/investigation.md` for detailed investigation patterns. Build multiple plausible hypotheses when meaningful; define a falsifiable prediction for each. Trace chronological execution including async boundaries, closures, effects, queries, subscriptions, user interactions and data consistency. Use discriminating evidence to reject alternatives, not just confirm the favorite explanation. Distinguish triggering conditions, proximal defect and systemic factors. For a claimed confirmed cause, show the causal chain `trigger → state/event → wrong decision → symptom`, linked to real code and/or observed runtime evidence. If not proven, mark **Likely** or **Unknown** explicitly.

## Phase 4 — Teach the diagnosis and pause
Explain why the code exists, what problem it originally solves, why the failure occurs, and when it occurs. Use before/after event timelines and small synthetic inputs with intermediate state; explain concepts such as stale closures or races in plain English (or the user's preferred language). Show the smallest counterexample. Deliver evidence and open questions, then request the **Diagnosis gate** approval.

## Phase 5 — Compare fixes and get approval
Read `references/solution-review.md`. Compare a minimal fix against at least one credible alternative where alternatives actually exist. Evaluate correctness, safety, scope, complexity, maintenance, runtime and regression risk; explicitly consider whether a guard hides the symptom. Prefer the narrowest fix that restores the intended invariant. State files to change, tests to add, migration/backward-compatibility implications, and rollback plan. No speculative cleanup or unrelated refactor. Recommend one option with rationale; **STOP for Solution gate**.

## Phase 6 — Implement approved scope and test
After explicit approval, preserve unrelated edits, write or adapt the smallest regression test that fails before the fix when feasible, then implement the approved change. Follow repo conventions, strict typing, schema validation at trust boundaries, authN/authZ separation, cancellation/cleanup, stable query keys and cache semantics, safe network timeouts/retries where relevant. Avoid extra dependencies without justification. Make the implementation resilient to empty/error/race/retry paths without broad architecture changes. Do not fabricate passing test output. Run targeted tests first, then relevant lint/typecheck/integration tests if available. If changing the plan becomes necessary, explain and request approval before expanding scope.

## Phase 7 — Independent code and regression review
Read `references/solution-review.md` and `references/report-template.md`. Review the *actual diff* as if authored by somebody else. Check: root-cause correction vs workaround, correctness/invariants, concurrency ordering, stale state, lifecycle cleanup, error and retry behavior, idempotency, security, type safety, API compatibility, performance, accessibility if UI, test design, and unrelated edits. Trace affected call sites and shared utilities to identify blast radius; verify that existing feature behavior is preserved. Classify findings `Critical/High/Medium/Low`, with concrete location, repro scenario, and suggested fix. Clearly state what was tested vs manually inspected vs unverified. Fix defects within approved scope; request approval for unrelated improvements. Present the **Review gate**.

## Phase 8 — Final report
Use `references/report-template.md` as the report structure. Include symptom, workflow summary, reproduction status, root-cause confidence and evidence, selected solution and rejected alternatives, exact change summary, test matrix including edge cases, review findings, residual risk, and next actions. Never claim “all edge cases tested”: describe the tested universe and gaps. Include actual commands/results if run, or `Not run (reason)`. End with a concise explanation of the lesson learned.

## Communication contract
Be structured but conversational. At the beginning of each phase give a 1–3 sentence objective. Explain code in the order it executes. Favor short concrete examples over theory and avoid dumping unchanged files. Use path:line and focused snippets for inspection, showing only changed lines for proposed edits. Prefer practical evidence over confident speculation. Preserve user control while avoiding redundant questions. If a user says “continue” at a gate, treat it as approval for the *next analysis phase* only, not code changes unless the solution gate explicitly requested implementation approval.
