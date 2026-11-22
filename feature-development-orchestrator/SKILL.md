---
name: feature-development
description: File-based, approval-gated workflow for building a new software feature across multiple chats, with state stored in `.ai/features/<slug>/`. Use when the user asks to start a new feature, be grilled/interviewed on its requirements, investigate a codebase for a feature, plan a feature into tasks, implement / review / approve a task ID (e.g. "implement T03"), resume or replan a feature, or do a final feature review. Do not use for one-off bug fixes or small edits that need no requirements or plan.
---

# Feature Development

Feature state lives in `.ai/features/<slug>/`, not in chat history. Start every session by reading these files. After every meaningful decision, write it to them.

```
.ai/features/<slug>/
  SKILL.md            # feature-local instructions (plain file, not an installed skill; read it explicitly)
  requirements.md
  architecture.md
  master-plan.md
  decisions.md        # append-only
  tasks/T01-<slug>.md
  handoffs/T01.md
```

## Step 0: Choose the operation and load state

1. Map the request to one operation: `discover`, `investigate`, `plan`, `implement Txx`, `review Txx`, `approve Txx`, `resume`, `replan`, or `feature review`. Ask one question only if the request is truly ambiguous.
2. Read the feature's `SKILL.md`, `requirements.md`, `architecture.md`, `master-plan.md`, `decisions.md`, the relevant task file, and its latest handoff. If several feature directories exist and the slug is unclear, ask which one.
3. Check the gate. If the gate isn't met, say which one is missing and offer the earlier step.

| Operation | Gate |
|---|---|
| investigate | requirements.md approved by user |
| plan | requirements approved; architecture.md exists |
| implement Txx | master plan approved; all Txx dependencies `approved` |
| review Txx | Txx is `in_review` |
| approve Txx | review done, no open blockers, explicit user "approve" |
| feature review | all tasks `approved` |

**Output:** one line stating the feature, the operation, and the current status.

## Step 1: Discover ("grill me")

1. Ask 3–5 related questions per round and wait for the answers. Cover topics in this order: problem and success measure, then actors and workflow, then business rules and permissions, then data and error cases, then non-goals.
2. After each round, play back a short trace with mock data. Example: "Alice (manager) refunds a $40 order, so the status becomes `refunded` and the customer is emailed." Ask the user to correct it.
3. Never invent business rules. Label every requirement with one of:
   - `CONFIRMED`: the user said it.
   - `ASSUMED`: your labeled default, for minor points only.
   - `OPEN`: needs a decision.
   - `REJECTED`: explicitly out of scope.
4. Write `requirements.md` and list the OPEN items that need decisions.

**Output:** `requirements.md`.
**Gate:** the user explicitly approves. Record the date in the file. Until then, keep asking.

## Step 2: Investigate the codebase

1. Record the branch and commit (`git rev-parse --abbrev-ref HEAD`, `git rev-parse --short HEAD`).
2. Read the code this feature will touch: entry points, data models, the most similar existing feature, its tests, and the relevant config.
3. Identify the smallest integration points, the patterns to reuse, side effects, and constraints (auth, transactions, migrations, external calls).
4. Mark every path and claim `VERIFIED` (you read it) or `TO VERIFY`. Do not change any code.

**Output:** `architecture.md`.

## Step 3: Plan

1. Split the work into a few tasks (usually 3–8). Define each task by a verifiable behavior or contract, not by file boundaries. A task must be doable in a fresh chat by someone who has read only the feature files and that task file.
2. Write `tasks/Txx-*.md`, `master-plan.md`, `decisions.md`, and the feature-local `SKILL.md`, and create `handoffs/`.
3. Include build and test commands only if you found them in the repo (package scripts, Makefile, CI config). Otherwise write `TO VERIFY`.

**Output:** the plan files.
**Gate:** the user approves the master plan. Record the date. Do not implement anything during planning.

## Step 4: Implement one task (Txx)

1. Re-read the feature files and the current source the task touches, since the code may have changed. If a dependency isn't `approved`, set the task to `blocked` and stop.
2. In 5 lines or fewer, state the objective, scope, approach, and main trade-off.
3. If the user proposed an approach, evaluate it before coding: its benefit, its downside, and how it could fail. Offer an alternative only if it is materially better. If the approach is unsafe or incorrect, say so before editing, and separate blockers from preferences.
4. Edit only within the task's scope and follow the repo's conventions. Integration fixes outside the scope are allowed only when necessary, and must be listed in the handoff. Make no unrelated refactors or speculative abstractions.
5. Where relevant:
   - validate external input and enforce permission checks;
   - set timeouts on network calls;
   - use transactions or idempotency where a partial failure could corrupt data.
6. Run focused checks: tests, type checks, and lint for the touched code. Mark each check `EXECUTED` with its result, or `NOT RUN` with the reason. Compiling is not runtime verification.
7. Write `handoffs/Txx.md` and set the status to `in_review` in both the task file and the master plan. Never set it to `approved` here.

**Output:** a diff summary and the handoff file.

## Step 5: Review one task (Txx)

1. Get the actual diff against the base commit recorded in the handoff (`git diff <base>`). Compare it with the acceptance criteria and requirements, not just with the plan.
2. Check these in order, skipping any that don't apply:
   1. Every acceptance criterion is met.
   2. Empty, error, and permission-denied paths are handled.
   3. Partial success and retries are handled.
   4. Concurrent writes to shared data are handled.
   5. Callers have no regressions.
   6. The tests actually exercise the behavior.

   Skip exotic cases unless their impact is high.
3. Report each finding as `blocker`, `important`, or `optional`, with its repro condition and a fix. List what was not tested.
4. Fix findings only when the user says to. Rerun the relevant checks and append the results to the handoff.

**Gate:** only explicit user approval sets the task to `approved` in the task file and the master plan. Passing tests are not approval. If the user rejects the work, set the task to `in_progress` and record the reasons.

## Step 6: Resume, replan, finish

- **Resume:** run Step 0. Compare `git status` and `git log` with the latest handoff, report any discrepancies, and propose the next unblocked task.
- **Replan:** explain the impact on approved requirements, tasks, and dependencies. Change approved scope only after the user approves. Log the change in `decisions.md` and mark affected tasks `needs_replanning`.
- **Feature review:** check the work end to end: API and data contracts across tasks, migrations, UI states, permissions, and whether the tests cover the requirements.
  - **Gate:** the user approves completion.
  - Never mark the feature complete while blockers remain open or required acceptance criteria are unverified.

## Status and records

- Task statuses are `not_started`, `in_progress`, `blocked`, `in_review`, `needs_replanning`, and `approved`. A status is not evidence; the handoff holds the evidence.
- Never silently change a CONFIRMED requirement or an approved decision.
- With multiple agents, one agent owns all writes to `master-plan.md`. Each other agent writes only its own handoff file.
- Record commit or PR links only when they are known. Never invent them.

## Edge cases

- **No repository access:** run discovery normally. Mark the architecture and plan `PROVISIONAL`, with every claim `TO VERIFY`. Do not implement or review, and never fabricate code, APIs, or test results.
- **No file-writing tools:** output each file's full content under its path and say the files are not saved. In the next session, ask the user to provide or confirm them.
- **Feature directory not found:** list `.ai/features/`. If it is empty, ask whether to start discovery or have the user provide the files.
- **Files disagree with the code:** trust the code for the current state and the approved files for intent. Report the gap instead of reconciling it silently.
- **No test commands found:** say so, and list the suggested commands as `NOT RUN`.

## Ending a session

Give the user this starter to copy:
`Use the feature-development skill. Resume .ai/features/<slug>/ and implement Txx. Read its SKILL.md, requirements.md, architecture.md, master-plan.md, tasks/Txx-*.md, and latest handoff. Verify the current code before editing.`

## Templates

**requirements.md**
```
# <Feature>: Requirements
Status: draft | approved <date>
## Problem & success measure
## Actors
## Workflow (example trace with mock data)
## Requirements
| ID | Requirement | State (CONFIRMED/ASSUMED/OPEN/REJECTED) | Notes |
## Non-goals
## Open decisions
```

**architecture.md**
```
# <Feature>: Architecture
Branch / commit:
## Integration points (path | role | VERIFIED/TO VERIFY)
## Patterns to reuse
## Data / contract changes
## Constraints & risks
```

**master-plan.md**
```
# <Feature>: Master plan
Approved: no | <date>
| Task | Behavior | Depends on | Status |
## Verified commands
## Risks
```

**tasks/Txx-<slug>.md**
```
# Txx: <behavior>
Status: not_started
Depends on: | Requirements: R1, R3
## Objective
## Scope (in / out)
## Acceptance criteria
- Given … When … Then …
## Tests
## Risks
```

**handoffs/Txx.md**
```
# Txx handoff: <date>
Base commit:
## Changes (file: why)
## Out-of-scope integration fixes
## Checks (EXECUTED/NOT RUN: result)
## Deviations & open issues
## Review findings (blocker/important/optional)
```

**decisions.md**
```
## Dxx: <title> (<date>)
Context: | Decision: | Alternatives: | User-approved: yes/no
```

**Feature-local SKILL.md**
```
# <Feature>: agent instructions
Workflow and gates: follow the feature-development skill.
Read in order: requirements.md, architecture.md, master-plan.md, tasks/Txx-*.md, handoffs/Txx.md.
Feature rules: <conventions, areas not to touch, verified commands>
```