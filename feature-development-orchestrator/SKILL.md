---
name: feature-development
description: Orchestrate new software features from discovery through approval using a reusable, file-based workflow. Use when a developer wants to be grilled on business requirements, investigate a codebase, generate a feature-specific SKILL.md and independent task plans, implement a planned subtask in a fresh chat, critically review an implementation, resume progress, or finish a feature. Maintain durable requirements, decisions, task status, and evidence instead of relying on conversation history.
---

# Feature Development Orchestrator

Run a **file-first, approval-gated** feature lifecycle. The reusable skill is the process; each generated `.ai/features/<feature-slug>/` directory is the feature-specific source of truth. A feature-local `SKILL.md` is an instruction file for agents reading that repository; it is **not automatically installed or discoverable** as a global skill. Load it explicitly in a fresh session unless the host supports repository-local skills.

## Operating rules

1. First resolve the requested operation: `new feature` / `grill me` / `investigate` / `plan` / `implement <task-id>` / `review <task-id>` / `approve <task-id>` / `resume` / `replan` / `feature review`. If ambiguous, inspect feature files and ask one targeted question only when necessary.
2. If a repository/workspace is available, read the existing feature files and relevant source before acting. If inaccessible, say so; don't fabricate codebase findings, APIs, test results, or file changes. You may provide draft artifacts clearly marked as provisional.
3. Record outcomes in the feature directory after each meaningful decision. **Durable files, not chat history, govern resumed work.** Respect prior user approvals and never silently change confirmed requirements.
4. Separate `CONFIRMED`, `ASSUMED`, `OPEN`, and `REJECTED` requirements. Ask for decisions on consequential unknowns; propose sensible, labeled defaults on minor ones. Do not fill in missing business rules as facts.
5. Preserve user control. Require explicit approval of requirements before final planning, of the master plan before implementation, and of reviewed work before marking a task `approved`. Never claim user approval from a passing test.
6. Prefer simple, established repository patterns. Avoid unrelated refactors and speculative abstractions. Validate external inputs and permission boundaries; prefer typed errors, idempotent mutations where appropriate, timeouts for network calls, and verified transactions/indexes where relevant. Do not force stack-specific patterns into other stacks.
7. Be constructive but critical of proposed implementations: explain benefit, downside, meaningful failure modes, and an alternative **only if materially better**. If a requested approach is unsafe/incorrect, identify that before editing; distinguish blockers from preferences.
8. Do not implement the entire feature during planning, or implement unapproved tasks. Make changes only within agreed task scope except necessary integration fixes, which must be explained and recorded.
9. Review important edge cases by **likelihood x impact**; include common empty/error/permission/race/partial-success paths when relevant. Avoid exotic edge cases unless impact is high. Never equate compilation with runtime verification.
10. Keep generated files concise, explicit, and linked by stable task IDs. Prefer a few coherent tasks over dozens of tiny edits. Define each task by a verifiable behavior or contract, not by arbitrary file boundaries.

## Lifecycle and phase gates

### 1 — Grill me / discover

Use `references/discovery.md`. Ask 3–5 related questions per round, waiting for answers before the next. Begin with problem, actors, workflow, and success; then explore business rules, auth, data, errors, boundaries, and non-goals. Show a short example/trace to confirm shared understanding. Summarize into `requirements.md` using `assets/requirements.template.md`. Present unresolved decisions. **Gate:** user approves the requirements baseline; otherwise remain in discovery.

### 2 — Investigate the codebase

Read relevant existing code, documentation, contracts, tests, and current repository state. Find the smallest integration points, reusable patterns, side effects, and constraints. Create `architecture.md` using `assets/architecture.template.md`. Mark every file path and claim as `VERIFIED` or `TO VERIFY`; note commit/branch when available. If the repository is absent, stop short of asserting a final technical design. Do not refactor anything during investigation.

### 3 — Plan and generate a feature-local skill

Read `references/planning.md`, and generate `.ai/features/<feature-slug>/SKILL.md` from `assets/feature-skill.template.md`, `master-plan.md` from `assets/master-plan.template.md`, and `tasks/NN-<slug>.md` from `assets/task.template.md`. Also maintain `decisions.md` and `handoffs/`.

Use task IDs `T01`, `T02`, etc. List dependencies, acceptance criteria, testing, actual integration boundaries, risks, and status. Every task must be executable with a fresh context **after reading the feature-local SKILL.md, requirements, architecture, master-plan, and that task**; avoid dependence on earlier chat. Include explicit commands only when verified in the repository. **Gate:** user approves the master plan before implementation.

### 4 — Implement one task

Read `references/implementation.md`. Reload the feature-local instructions and relevant up-to-date source. Confirm dependencies and current state. Summarize objective, scope, approach, trade-offs. If the user proposes implementation logic, critically evaluate it before coding. Make focused changes only for the selected task. Run available focused checks; distinguish executed checks from suggested checks. Record deviations and evidence in a handoff file. Status -> `in_review`, **not** `approved`.

### 5 — Review one task

Read `references/review.md`. Compare actual diff against acceptance criteria and business rules, not merely the plan text. Rank findings as `blocker`, `important`, or `optional` and include precise repro conditions and fixes. Verify security, errors, races and regressions proportionally. Report what was not tested. Address findings upon user instruction; rerun relevant checks. **Gate:** only user approval moves task to `approved` (and updates master plan). If rejected, remain `in_progress`/`in_review` with reasons.

### 6 — Handoff, resume, integrate

For `resume`, identify the feature directory, read current plan and handoff files, check repository state, and propose the next unblocked task. For `replan`, explain the impact and obtain approval before changing approved scope or dependencies. For feature completion, perform an end-to-end integration review spanning tasks, API/data contracts, migrations, UI states, permissions, and tests. **Gate:** user approves feature completion. Never mark it complete with outstanding blockers or unverified required acceptance criteria.

## Status and update discipline

- Task status: `not_started`, `in_progress`, `blocked`, `in_review`, `needs_replanning`, `approved`. Status is not a substitute for evidence.
- Update the relevant task file, `master-plan.md`, `decisions.md` (for decisions), and `handoffs/Txx.md` (for implementation/review evidence). In multi-agent environments, serialize master-plan writes or nominate one owner; agents can write separate handoff files.
- Preserve history of decisions and newly discovered constraints. Record links to commits/PRs when known. Do not invent them.
- When ending a session, give a copyable fresh-chat starter: `Use the feature-development skill. Resume .ai/features/<slug>/ and implement Txx. Read its SKILL.md, requirements.md, architecture.md, master-plan.md, tasks/Txx-*.md, and latest handoff. Verify the current code before editing.`

## Output quality

Use concrete examples and small mock data to clarify business flows. Prefer practical acceptance criteria (`Given / When / Then`) and explicit success/failure states. Explain choices and trade-offs without lengthy generic architecture lectures. Honor local project conventions after inspection. If file-writing tools are unavailable, output the proposed files for the user to save rather than claiming they exist.
