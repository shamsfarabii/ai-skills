---
name: systematic-debugging
description: Evidence-driven debugging for an existing codebase. Use when asked to investigate or fix a bug, unexpected behavior, regression, race condition, duplicate request, stale UI, or performance issue. Trace the relevant code path, reproduce the issue, prove the root cause, propose a focused fix, get approval before editing, then fix and verify (with tests if the project has them, manually if not).
---

# Systematic Debugging

## Goal
Find the real cause of the bug, fix it with the smallest correct change, and prove the fix works. Don't just make the symptom disappear.

## Ground rules
- **Never fake evidence.** Don't claim you read a file, ran a test, or proved a cause unless you did. Cite real `path:line` locations and real command output.
- **Label confidence** on every claim: **Confirmed** (proven by code or runtime evidence), **Likely** (strong but unproven), or **Unknown**.
- **Read-only until approved.** Inspect code, run safe commands, and run local tests freely. Do not edit code until the user approves the fix.
- **Be safe.** Run `git status` first and keep the user's existing changes. No destructive commands, production writes, or exposed secrets.
- **Ask only when blocked.** If a missing detail blocks progress, ask one specific question. Otherwise state your assumption and keep going.

---

## Step 1 — Understand the code path
Trace only the path the bug travels through: entry point → component/handler → state → API/service → storage → output. Follow real call sites, not names or comments.

Note anything that commonly causes bugs along the way: async calls, effects, caching, retries, timers, listeners, cleanup, feature flags.

**Check for a test setup.** Look for a test runner config or script (for example, a `test` script in `package.json`, `jest`/`vitest`/`pytest` config, or a `tests/` or `__tests__/` folder). Note in one line whether tests exist. If they don't, skip every test step below and use manual verification instead. Don't add a test framework unless the user asks.

**Output (keep it short):**
- 2–4 sentences on what the feature should do
- The execution path as a numbered list or small diagram, with `path:line` refs
- Test setup: found / not found

## Step 2 — Reproduce
- Write **Expected vs Actual** in one line each.
- Find the minimal steps, environment, and frequency (always / sometimes / once).
- Reproduce it:
  - **If tests exist:** write a failing test. It becomes your regression test later.
  - **If no tests:** reproduce it with clear manual steps, a small script, logs, or a network trace. Write down the exact steps so you can repeat them after the fix.
- If you can't reproduce it, say **Not reproduced**, explain why, and continue from code evidence with that caveat.

## Step 3 — Find the root cause
1. List 2–3 plausible hypotheses. For each, write what you'd expect to see if it were true.
2. Check the evidence that tells them apart: logs, breakpoints, a targeted test, or a git diff/bisect if it's a regression. Rule hypotheses out; don't just look for support for your favorite.
3. Write the causal chain:
   `trigger → wrong state/event → wrong decision → symptom`
   Link each link to code or runtime evidence.

**Common culprits to check:** stale closures, race conditions or out-of-order responses, missing cleanup or cancellation, duplicate effect or listener registration, wrong cache keys or missing invalidation, null/empty/error paths, off-by-one errors, timezone or locale handling, type coercion, environment or config differences.

> If the cause is still only **Likely** or **Unknown**, stop and tell the user what you found, what's unclear, and what you'd check next. Ask whether to keep investigating.

## Step 4 — Propose the fix ⛔ STOP for approval
Present briefly:
- **Root cause** in plain language, with a tiny before/after example or timeline.
- **Recommended fix**: the smallest change that restores the correct behavior. Say which files change, and either the test you'll add or, if there are no tests, how you'll verify it manually.
- **Alternative**, if a real one exists, with one line on why you didn't choose it.
- **Risk**: what else uses this code, and what could break.

Ask: **"Should I implement this fix?"**
A vague "ok" or "continue" is not approval to edit. Get a clear yes.

Watch out for fixes that only hide the symptom, such as an extra guard, a debounce, or a try/catch that swallows the error. Fix the cause unless there's a stated reason not to.

## Step 5 — Fix and verify
1. **Before the fix:**
   - **If tests exist:** write or keep the failing test from Step 2 and confirm it fails.
   - **If no tests:** skip this. Keep your manual repro steps ready.
2. **Apply only the approved change.** Follow repo conventions. No unrelated refactors and no new dependencies without a reason.
3. **Verify:**
   - **If tests exist:** run the targeted test first, then related tests, lint, and typecheck if available.
   - **If no tests:** repeat the manual repro steps and confirm the bug is gone. Then check one or two nearby behaviors to make sure they still work. Still run lint, typecheck, or build if the project has them.
4. **Review your own diff** as if someone else wrote it:
   - Does it fix the cause, or just hide the symptom?
   - Does it handle empty, error, retry, and concurrent cases?
   - Are other callers of the changed code still correct?
   - Are there any leftover debug logs or unrelated edits?
5. If the fix needs to grow beyond the approved scope, stop and ask first.

## Final summary
Keep it short:
- **Bug:** expected vs actual
- **Root cause:** one or two sentences, with confidence level and `path:line`
- **Fix:** what changed and why
- **Verification:** test commands and results, or "No test setup — verified manually:" followed by the steps you ran and what you saw
- **Not covered:** remaining risks or untested cases
- **Lesson:** one sentence on what caused this class of bug

Never say "all edge cases tested." Say what was verified and what wasn't.