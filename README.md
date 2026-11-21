# ai-skills

Reusable [Cursor Agent Skills](https://cursor.com/docs) for software engineering workflows. Install the skills you need locally; invoke them by name or by describing the situation in chat so the agent can load the right `SKILL.md`.

## Which skill should I use?

Pick by **what you are trying to do right now**, not by file type.

| You want to… | Use this skill | Path |
|--------------|----------------|------|
| Write, review, refactor, or design code with consistent professional standards (SOLID, DRY, KISS, modularity, deep modules, least astonishment) | **Engineering principles** | [engineering-principles/SKILL.md](engineering-principles/SKILL.md) |
| Understand what a snippet, function, hook, component, class, query, or file does — with traces, mock data, trade-offs, and edge cases | **Code explanation tracer** | [code-explaination-tracer/SKILL.md](code-explaination-tracer/SKILL.md) |
| Run a full feature from discovery → requirements → codebase investigation → planning → one-task implementation → review → resume (file-based, approval-gated) | **Feature development orchestrator** | [feature-development-orchestrator/SKILL.md](feature-development-orchestrator/SKILL.md) |

### Quick decision guide

```
Need to SHIP a new capability end-to-end with docs and task plans?
  → feature-development-orchestrator

Need to LEARN or AUDIT existing code (what/why/how/data flow)?
  → code-explaination-tracer

Need to WRITE or REVIEW code quality, structure, or design choices?
  → engineering-principles
```

You can combine skills in one session when it helps — for example, use **engineering principles** while implementing a task planned under **feature development**, or use **code explanation tracer** before refactoring unfamiliar code.

---

## Skill reference

### Engineering principles

**Use when:**

- Implementing or changing production code and you want SOLID, simplicity, and maintainable boundaries.
- Reviewing a PR or diff for design smells (duplication, shallow modules, train-wrecks, speculative abstractions).
- The user asks for “clean code,” coding standards, or how to structure a module.
- You feel **friction** in the design: rigidity, surprise, copy-paste rules, or interfaces that invite misuse.

**Do not use when:**

- The main goal is only to narrate what existing code does (use **code explanation tracer**).
- The main goal is product discovery and multi-phase feature delivery (use **feature development orchestrator**).

**Extras:** Deeper principle notes live in [engineering-principles/reference.md](engineering-principles/reference.md).

---

### Code explanation tracer

**Use when:**

- The user asks what code does, why it exists, or how data flows through it.
- You need a step-by-step trace with realistic mock inputs and intermediate values.
- You want alternatives, trade-offs, and likely failure paths explained in plain language.
- Scope is a **specific** piece of code (snippet, function, hook, component, class, query, algorithm, or file) — any language.

**Do not use when:**

- The user wants you to **change** the code or enforce repo-wide standards (use **engineering principles** or project rules).
- The user wants a full feature lifecycle with requirements files and task gates (use **feature development orchestrator**).

---

### Feature development orchestrator

**Use when:**

- Starting a **new feature** and you want structured discovery (“grill me”), requirements, and explicit approvals.
- You need codebase **investigation**, architecture notes, a master plan, and independent **task files** under `.ai/features/<feature-slug>/`.
- You will **implement one task at a time** in fresh chats, **review** tasks against acceptance criteria, or **resume** after a break.
- Conversation history is unreliable — durable files should own requirements, decisions, status, and evidence.

**Typical phrases to trigger it:**

- `new feature`, `grill me`, `investigate`, `plan`, `implement T01`, `review T02`, `approve T03`, `resume`, `replan`, `feature review`

**Do not use when:**

- You only need a quick explanation of existing code (use **code explanation tracer**).
- You only need general design advice on a small edit (use **engineering principles**).

**Note:** This skill creates a **feature-local** `SKILL.md` inside your project (e.g. `.ai/features/my-feature/SKILL.md`). That file is instructions for agents working on that feature; it is **not** auto-installed as a global skill — load it explicitly when resuming work.

---

## Setup on Cursor, Claude, and Codex

Every skill in this repo is a **folder with a `SKILL.md`** (plus optional `reference.md`, `agents/openai.yaml`, and so on). Install by copying or symlinking the whole folder into the paths your tool scans. Symlinks are supported on all three platforms.

Set the repo path once (change it if you cloned elsewhere):

```bash
export AI_SKILLS_REPO="/Volumes/Saviour/Projects/ai-skills"
```

| Skill folder in this repo | Suggested install folder name | Slash / `$` command name |
|---------------------------|-------------------------------|---------------------------|
| `engineering-principles/` | `engineering-principles` | `engineering-principles` |
| `code-explaination-tracer/` | `code-explanation-tracer` | `code-explanation-tracer` (see `name` in frontmatter) |
| `feature-development-orchestrator/` | `feature-development` | `feature-development` |

> **Folder name vs `name` in frontmatter:** Claude Code uses the **directory name** as `/command`. Codex and Cursor also key off the folder containing `SKILL.md`. Use the suggested names above so invocations match the `name` field in each skill’s YAML frontmatter.

### One-shot install (all tools, user/global scope)

```bash
export AI_SKILLS_REPO="/Volumes/Saviour/Projects/ai-skills"

mkdir -p "$HOME/.cursor/skills" "$HOME/.agents/skills" "$HOME/.claude/skills" "$HOME/.codex/skills"

for pair in \
  "engineering-principles:engineering-principles" \
  "code-explaination-tracer:code-explanation-tracer" \
  "feature-development-orchestrator:feature-development"
do
  src="${pair%%:*}"; dst="${pair##*:}"
  ln -sf "$AI_SKILLS_REPO/$src" "$HOME/.cursor/skills/$dst"
  ln -sf "$AI_SKILLS_REPO/$src" "$HOME/.agents/skills/$dst"
  ln -sf "$AI_SKILLS_REPO/$src" "$HOME/.claude/skills/$dst"
  ln -sf "$AI_SKILLS_REPO/$src" "$HOME/.codex/skills/$dst"
done
```

Restart the app or start a new agent session after installing. Skills are discovered at startup.

---

### Cursor

**Docs:** [Agent Skills](https://cursor.com/docs/skills)

| Scope | Directory |
|-------|-----------|
| This project only | `<repo>/.cursor/skills/<skill-name>/` or `<repo>/.agents/skills/<skill-name>/` |
| All projects (this machine) | `~/.cursor/skills/<skill-name>/` or `~/.agents/skills/<skill-name>/` |

Cursor walks these trees recursively, so category subfolders (for example `~/.cursor/skills/workflows/feature-development/`) are fine.

**Compatibility:** Cursor also loads skills from `~/.claude/skills/`, `~/.codex/skills/`, and the matching project paths — so a single symlink under `~/.agents/skills/` often covers Cursor and Codex together.

**Project install (commit skills with a repo):**

```bash
cd /path/to/your-app
mkdir -p .cursor/skills
ln -sf "$AI_SKILLS_REPO/engineering-principles" .cursor/skills/engineering-principles
```

**Verify:** Sidebar → **Customize** → **Skills**. You should see each skill under **Agent Decides**.

**Run a skill:**

- Type `/` in Agent chat and pick the skill for one message, or use **Option+Enter** (Mac) / **Alt+Enter** (Windows) to keep it on for the session.
- Or describe the task; the agent matches skills from the `description` in frontmatter.

**Cloud Agents:** Personal skills in `~/.cursor/skills/` stay local unless you enable **Settings → Agents → Sync Skills for Cloud Agents**. For team-wide or CI automations, put skills in the repo’s `.cursor/skills/` instead.

**Optional CLI** (when this repo is published on GitHub): `npx skills add <owner>/<repo> --skill <name> -a cursor` with `-g` for global install into `~/.agents/skills/`.

---

### Claude (Claude Code)

**Docs:** [Extend Claude with skills](https://code.claude.com/docs/en/skills)

| Scope | Directory |
|-------|-----------|
| All projects (this machine) | `~/.claude/skills/<skill-name>/SKILL.md` |
| Shared with the team | `<repo>/.claude/skills/<skill-name>/SKILL.md` (commit to git) |
| Monorepo / subpackage | `<subdir>/.claude/skills/<skill-name>/` (loaded when working under that tree) |

**Example:**

```bash
mkdir -p "$HOME/.claude/skills"
ln -sf "$AI_SKILLS_REPO/engineering-principles" "$HOME/.claude/skills/engineering-principles"
```

**Run a skill:** In the Claude Code terminal, type `/engineering-principles`, `/code-explanation-tracer`, or `/feature-development`, or ask in natural language and Claude loads the skill when the description matches.

**Claude on the web (Cowork / claude.ai):** Upload or enable skills in your account settings; filesystem paths above apply to **Claude Code on your machine**, not automatically to every cloud product. See [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

---

### Codex (OpenAI Codex CLI / IDE extension)

**Docs:** [Build skills](https://learn.chatgpt.com/docs/build-skills) (Codex customization)

| Scope | Directory |
|-------|-----------|
| User (any repo) | `~/.agents/skills/<skill-name>/` |
| Repo root (team) | `$REPO_ROOT/.agents/skills/<skill-name>/` |
| Codex home (installer default) | `~/.codex/skills/<skill-name>/` |

Codex scans `.agents/skills` from the current working directory up to the git root. Nested folders can each host their own `.agents/skills/` for scoped skills.

This repo already ships **`agents/openai.yaml`** in each skill for Codex/ChatGPT UI metadata (display name, short description). No extra config is required for basic use.

**Example:**

```bash
mkdir -p "$HOME/.agents/skills"
ln -sf "$AI_SKILLS_REPO/feature-development-orchestrator" "$HOME/.agents/skills/feature-development"
```

**Run a skill:** In Codex, use `/skills` or mention `$feature-development` (and similar) in the prompt.

**Install from GitHub (Codex built-in installer):** When this repository is on GitHub, you can use Codex’s `$skill-installer` or the installer scripts documented in [OpenAI’s skill-installer sample](https://github.com/openai/codex/blob/main/codex-rs/skills/src/assets/samples/skill-installer/SKILL.md) to copy skills into `~/.codex/skills/`.

**Optional CLI:** `npx skills add <owner>/<repo> --skill <name> -a codex` (add `-g` for user-level).

---

### Invoking skills (any platform)

Examples:

- *“Use the engineering-principles skill and review this diff.”*
- *“Use code-explanation-tracer on `src/hooks/usePortfolio.ts`.”*
- *“Use feature-development. Grill me on a new export-to-CSV feature, then investigate the repo.”*

After installation, agents also match skills from the YAML **`description`** when your request fits that scenario.