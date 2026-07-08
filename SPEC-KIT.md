# Spec-Driven Development with Spec-Kit (Claude + v0)

A **portable instructions file** — drop this into any project that uses the MomentMan4 Spec-Kit
setup. It's everything you need in one place: requirements, install, the workflow, how v0 fits,
and how to reuse the setup across projects. (The repo `README.md` is the SpecKit repo's landing
page; this file is meant to travel with a project.)

With Spec-Driven Development (SDD), the **spec is the source of truth**: write a spec → generate
a plan → break it into tasks → let the agent implement against them, instead of prompting
ad-hoc.

---

## 1. Requirements

| Requirement | Needed | Why |
|---|---|---|
| Python 3.11+ | ✅ | Runs the `specify` CLI |
| uv (or pipx) | ✅ | Installs `specify-cli` |
| Git | ✅ | Spec-Kit is git-aware (a feature branch per spec) |
| Claude Code | ✅ | Runs the `/speckit-*` slash commands |
| v0 account (Vercel) | optional | UI generation (see §5) |

You install `specify-cli` **once per machine**. After a project is initialized, the CLI is not
needed again — the committed `.claude/` skills and `.specify/` templates run the whole workflow.

---

## 2. Install `specify-cli` (once per machine)

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

Alternative (no uv): `pipx install` from the same git URL.

---

## 3. Set up a project

**Option A — `specify init` (recommended, always current):**

```bash
cd <your-project>
specify init --here --integration claude --force
# add --ignore-agent-tools if the `claude` binary isn't on PATH
```

**Option B — copy from the SpecKit repo (pins these exact versions):**

```bash
git clone --depth 1 https://github.com/MomentMan4/SpecKit /tmp/speckit
cp -r /tmp/speckit/.claude/skills /your-project/.claude/skills   # merge, don't clobber other skills
cp -r /tmp/speckit/.specify /your-project/.specify
```

Then seed standards: copy `.specify/memory/constitution.md` from the SpecKit repo and adjust the
two `[PROJECT-SPECIFIC]` spots (backend/auth, data layer). Commit `.claude/` and `.specify/`.
Secrets stay in `.env*` (already gitignored) — never commit keys.

### What the setup adds
```
.claude/skills/speckit-*/SKILL.md   # the /speckit-* slash commands
.specify/
  memory/constitution.md            # project principles (source of truth)
  templates/                        # spec, plan, tasks, checklist templates
  scripts/                          # helper scripts the commands call
  workflows/                        # workflow registry
```

---

## 4. The workflow (run as Claude Code slash commands)

Run in order. Each command reads the artifacts the previous one produced.

| Step | Command | Produces / does |
|---|---|---|
| 1 | `/speckit-constitution` | Establish/adjust project principles & guardrails |
| 2 | `/speckit-specify` | The **what & why**: user stories + acceptance criteria (no tech detail) |
| 3 | `/speckit-clarify` *(optional)* | Structured questions to de-risk ambiguity **before** planning |
| 4 | `/speckit-plan` | The **how**: architecture & stack decisions |
| 5 | `/speckit-tasks` | Ordered, independently shippable task list |
| 6 | `/speckit-analyze` *(optional)* | Cross-artifact consistency report |
| 6 | `/speckit-checklist` *(optional)* | Quality checklist for requirement completeness |
| 7 | `/speckit-implement` | Executes the tasks to build the feature |
| — | `/speckit-converge` | Assess existing codebase, append remaining work as tasks |
| — | `/speckit-taskstoissues` | Turn the task list into GitHub issues |

**Golden rule:** the spec owns *what/why*, the plan owns *how*. Don't put implementation detail
in the spec, and don't invent scope during implementation — update the spec instead.

See `specs/001-waitlist-email-capture/spec.md` in the SpecKit repo for a worked example spec.

---

## 5. Using SpecKit with Claude and v0 — step by step

**The division of labor:** **v0 (Vercel)** authors and iterates the **UI**; **Claude Code +
Spec-Kit** owns the **engineering** — spec, plan, structure, logic, data, integration, tests —
driven by the spec's tasks. The constitution encodes this split, so every `/speckit-plan`
respects it. Here is the full loop.

### Phase 0 — One-time setup (Claude Code)

1. Install the CLI once per machine: `uv tool install specify-cli --from git+https://github.com/github/spec-kit.git`
2. In the project: `specify init --here --integration claude --force`
3. Copy `.specify/memory/constitution.md` from the SpecKit repo and adjust the `[PROJECT-SPECIFIC]`
   spots (backend/auth, data layer).
4. Run **`/speckit-constitution`** — review the principles and **declare the project type**
   (Application / Marketing website / Website).

### Phase 1 — Spec & plan in Claude (before any UI)

5. **`/speckit-specify`** — describe the feature's *what & why* + acceptance criteria. Answer the
   **compliance & privacy triage** (health/finance/PII/children/AI). This is where you also state
   any AI capability's use case, scope, and industry.
6. **`/speckit-clarify`** *(optional)* — let Claude ask targeted questions to remove ambiguity
   **before** planning.
7. **`/speckit-plan`** — Claude produces the *how*: architecture, data model, auth, the pinned
   AI model/provider (if any), the risk assessment, and the data classification.
8. **`/speckit-tasks`** — ordered, independently shippable tasks (including test, a11y, SEO, and
   compliance tasks).
9. **`/speckit-analyze`** and, for escalated/regulated projects, **`/speckit-checklist`** — verify
   the spec/plan/tasks are consistent and that compliance controls are covered.

> Outcome of Phase 1: `spec.md`, `plan.md`, `tasks.md` (and any checklist) under
> `specs/NNN-feature/`. **Now** you know exactly which screens/components the UI needs.

### Phase 2 — Design the UI in v0 (Vercel)

10. Open **v0.dev** and prompt it using the spec: paste the relevant user stories and the list of
    screens/components the plan calls for. Ask for the specific pieces (e.g. "hero with email
    capture form, footer CTA, success + error states"), not a whole app.
11. **Feed v0 your design rules** so its output matches the constitution: the stack is Next.js
    App Router + Tailwind + shadcn/Radix; use design tokens, a defined type scale, light + dark
    themes, and **all states** (default/hover/focus/disabled/loading/empty/error). v0 already
    targets this stack, which keeps the output compatible.
12. Iterate visually in v0 until the screens look right. Keep components **presentational** — v0
    builds the look; real data comes later in Claude.
13. Bring the code into the repo: either **push from v0 to the connected GitHub repo/branch**, or
    copy the generated components into `components/`. Commit on a feature branch.

### Phase 3 — Integrate & implement in Claude

14. Back in Claude Code, **`/speckit-implement`** — Claude executes the tasks: it wires the
    v0 components to real data and server actions, adds validation/auth, replaces mock content,
    and builds the logic the spec calls for. It **preserves v0's visual intent and Tailwind
    classes** while moving behavior into hooks/actions/`lib` (per Principle II).
15. As it implements, Claude writes the **tests** the tasks call for and keeps
    `build`/`lint`/`typecheck`/tests green.

### Phase 4 — QA, review, and ship

16. **`/verify`** — drive the actual flow end-to-end (not just "the build passed").
17. **`/code-review`** and, for anything touching data/auth/compliance, **`/security-review`** —
    a second set of eyes before merge.
18. Open a PR: **CI runs the gates** (build, lint, typecheck, tests) and **Vercel builds a preview**
    — review the UI on the preview link.
19. Merge only when the **Definition of Done** is met (green CI, reviewed, a11y + responsive,
    SEO/metadata for marketing, AI evals for AI features, compliance sign-off if escalated).
    Vercel ships production from the default branch.

### The loop

For the next feature, go back to **Phase 1** (`/speckit-specify`). New UI → hop to v0 (Phase 2);
pure logic/refactors → stay in Claude. Same spec-driven backbone every time.

---

## 6. Notes & gotchas

- **Slash command names use hyphens** in this Spec-Kit version: `/speckit-specify`, not
  `/speckit.specify`.
- **Ephemeral environments:** in a fresh remote/cloud session the `specify` CLI won't persist,
  but the committed `.claude/` + `.specify/` files are enough to run the workflow. Re-install
  `specify-cli` only to `init` a brand-new project.
- **Keep `main` green**, work on feature branches; Vercel gives a preview per PR for visual
  review of v0-authored UI.

---

_Reference: <https://github.com/github/spec-kit> · Reusable setup: <https://github.com/MomentMan4/SpecKit>_
