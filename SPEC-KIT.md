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

## 3. Set up

Setup has two layers: **(a) add SpecKit to the project** (once per repo — the files that make the
`/speckit-*` commands exist), then **(b) open that project on whichever surface you're using**
(Claude Code desktop/local, Claude Code web, or v0). Then §3.5–3.6 cover what to do after running
the specs and how to keep everything synced.

### 3.1 Add SpecKit to the project (once per repo)

**Option A — `specify init` (freshest templates; needs Python 3.11+ & uv):**

```bash
cd <your-project>
specify init --here --integration claude --force
# add --ignore-agent-tools if the `claude` binary isn't on PATH
```

**Option B — copy from the SpecKit repo (no extra tools; pins your standards):**

```bash
git clone --depth 1 https://github.com/MomentMan4/SpecKit /tmp/speckit
mkdir -p ./.claude
cp -r /tmp/speckit/.claude/skills ./.claude/skills   # merge, don't clobber other skills
cp -r /tmp/speckit/.specify ./.specify
```

Then seed standards: copy `.specify/memory/constitution.md` from the SpecKit repo and adjust the
two `[PROJECT-SPECIFIC]` spots (backend/auth, data layer). **Commit and push** `.claude/` and
`.specify/` so every surface sees them. Secrets stay in `.env*` (already gitignored) — never
commit keys.

Starting a brand-new project? Clone SpecKit as the seed:
`git clone https://github.com/MomentMan4/SpecKit my-app && cd my-app && rm -rf .git && git init`.

**What this adds:**
```
.claude/skills/speckit-*/SKILL.md   # the /speckit-* slash commands
.specify/
  memory/constitution.md            # project principles (source of truth)
  templates/                        # spec, plan, tasks, checklist templates
  scripts/                          # helper scripts the commands call
  workflows/                        # workflow registry
```

### 3.2 Set up Claude Code — desktop app & local CLI

Desktop and CLI are the same "local Claude Code" — it loads slash commands from the open
project's `.claude/skills/`.

1. Make sure the project on disk contains `.claude/skills/speckit-*` (from §3.1).
2. **Open the project at its repo root** — desktop: *Open Project* and select the folder (the one
   that contains `.claude/`); CLI: `cd` into it and run `claude`.
3. Type `/` — the `speckit-*` commands should appear. Start with `/speckit-constitution`.
4. **If the commands don't show:** confirm `.claude/skills/speckit-constitution/SKILL.md` exists at
   the **repo root** (not a subfolder), that you opened the project folder itself (not a parent),
   then reopen the project / restart the session so it re-scans skills.

### 3.3 Set up Claude Code on the web (claude.ai/code)

The web environment is an ephemeral cloud container that **clones your repo fresh** each session.

1. In claude.ai/code, start a session on the **GitHub repo + branch** that already has
   `.claude/` + `.specify/` committed (from §3.1). The `/speckit-*` commands load automatically —
   no install needed.
2. Work as usual, then **commit and push before you finish** — the container is reclaimed after
   the session, so anything unpushed is lost.
3. *(Optional)* Add a `SessionStart` hook / setup script to the repo so each web session runs
   `pnpm install` (and, only if you plan to `specify init` from web, installs `specify-cli`). This
   makes the session build- and test-ready on boot. Outbound network follows the environment's
   network policy.

### 3.4 Set up v0 (web)

v0 is web-only and coordinates with the rest through **GitHub** — no `.specify` files needed on
v0's side; it just needs to read/write the same repo.

1. Sign in to **v0.dev** with your Vercel account.
2. **Connect GitHub** and point v0 at the project **repo and feature branch** (v0's GitHub sync /
   "Push to GitHub"). This is the whole integration — v0 pushes its work to that branch, which is
   exactly the hand-off Claude Code reads.
3. Give v0 your standards as context: paste the relevant part of the **constitution** (design
   rules, stack, "all states") and the spec's user stories into the prompt (or a v0 Project's
   custom instructions) so its output matches the constitution.
4. Deploy/preview is via Vercel, which is already the deploy target.

### 3.5 After you run the specs — what to do next

Once `/speckit-specify` → `/speckit-plan` → `/speckit-tasks` have produced `spec.md`, `plan.md`,
and `tasks.md` under `specs/NNN-feature/`:

1. **Commit the spec artifacts** (`specs/…`, updated `.specify/`) and push — they're the source of
   truth the implementer and reviewers rely on.
2. **Choose the implementer** for the feature (v0, Claude Code, or split — see §5 Phase 2) and
   record it in the plan.
3. **Build it** — `/speckit-implement` in Claude Code, and/or build in v0 — on a **feature
   branch**.
4. **QA & review** — `/verify`, then `/code-review` (+ `/security-review` for data/auth/compliance).
5. **Open a PR** — CI runs the gates, Vercel builds a preview; review the UI on the preview link.
6. **Merge on the Definition of Done** (green CI, reviewed, a11y/responsive, SEO for marketing, AI
   evals for AI features, compliance sign-off if escalated). Vercel ships production from the
   default branch.
7. **Next feature:** back to `/speckit-specify`. Keep the spec updated if scope changes mid-build —
   the spec, not the code, is the record.

### 3.6 Keeping your repos synced

All three surfaces share state **only through Git**, so treat it as the single sync point (full
rules in §6):

- **Push everything that matters, always** — especially from Claude Code web (ephemeral).
- **Pull before you start** on any surface; **push when you finish.**
- **One surface per branch at a time** — don't edit the same branch in v0 and Claude Code at once.
  Use the feature branch as the hand-off token (v0 pushes → Claude Code pulls, or vice-versa).
- **Commit the spec files too** (`.specify/`, `specs/`), not just code — that's how the next
  surface gets the plan and tasks.
- If two surfaces diverged, reconcile on the branch (pull/rebase) before continuing — don't let
  local, web, and v0 drift.

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

**How the tools relate:** **Spec-Kit** (run in Claude Code) owns the *thinking* — spec, plan,
tasks — for every project. **Implementation is a choice you make per feature:** both **v0** and
**Claude Code** are capable full-stack builders (v0 does UI *and* backend — routes, server
actions, data wiring — not just visuals), so you assign each feature to v0, to Claude Code, or to
a split. Whichever tool builds, the constitution's standards apply so the other can extend it.
Here is the full loop.

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
> `specs/NNN-feature/`. **Now** you know exactly what to build.

### Phase 2 — Choose the implementer (per feature)

10. Decide who builds each feature/task and record it in the plan. It can differ per feature and
    change over time:
    - **v0** — fast in-browser iteration, visual/preview-driven work, and full-stack scaffolds you
      want running quickly (marketing surfaces, CRUD screens, prototypes, straightforward server
      actions/routes).
    - **Claude Code** — complex or cross-cutting logic, large refactors, precise control, test
      authoring, tricky integrations/migrations, and anything driven from the repo/CLI.
    - **Split** — e.g. v0 builds the surface + scaffolds the endpoints; Claude Code hardens logic,
      security, and tests. Either tool can also own a whole feature end to end.

### Phase 3 — Implement (v0 and/or Claude Code)

11. **If v0 builds it:** prompt v0 with the relevant user stories and what the plan calls for, and
    **feed it the constitution's rules** — Next.js App Router + Tailwind + shadcn/Radix, design
    tokens, a defined type scale, light + dark themes, **all states**
    (default/hover/focus/disabled/loading/empty/error), and — for backend work — validation, auth,
    and a typed data layer (not the DB called straight from a component). Iterate, then **push from
    v0 to the connected GitHub branch** (or copy the code into the repo) on a feature branch.
12. **If Claude Code builds it:** run **`/speckit-implement`** — Claude executes the tasks across
    the stack, writes the tests the tasks call for, and keeps `build`/`lint`/`typecheck`/tests
    green.
13. **Hand-offs go both ways.** When Claude Code extends v0's output (or vice-versa), the second
    tool **preserves the first's intent** — visual design, Tailwind classes, existing structure —
    and finishes to constitution standard: real data wired (no leftover mock content), behavior in
    hooks/actions/services/`lib`, presentational components kept dumb. This clean-handoff contract
    is what lets either tool pick up the other's work without a rewrite.

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

For the next feature, go back to **Phase 1** (`/speckit-specify`), then **Phase 2** — pick the
implementer that fits *this* feature (v0, Claude Code, or split). Same spec-driven backbone every
time, regardless of who builds.

---

## 6. Working across surfaces — Claude Code (local + web) and v0 (web)

**Short answer: SDD works the same across all of them, because the spec is in Git.** The spec,
plan, tasks, constitution, and the `/speckit-*` skills all live in the repo (`.specify/`,
`specs/`, `.claude/`). Every surface — Claude Code on your machine, Claude Code on the web
(claude.ai/code), and v0 on the web — reads and writes that same repo, so the method doesn't
change. What changes is **coordination**: treat **Git as the single sync point** and mind that the
web Claude Code environment is **ephemeral**.

**Rules that make multi-surface work reliable:**

1. **Git is the source of truth — commit and push everything that matters.** Spec artifacts
   (`specs/`, `.specify/`, `.claude/`) and code must be committed so the next surface picks them
   up. On **Claude Code web**, the cloud environment is reclaimed after the session — anything not
   pushed is lost. On **local**, it persists on disk, but still push so web and v0 see it.
2. **Pull before you start, push when you finish.** Before continuing a feature on a different
   surface, pull the latest for that branch; when done, push. This is what keeps local ↔ web ↔ v0
   in sync.
3. **One surface per branch at a time.** Don't edit the same branch simultaneously in v0 and
   Claude Code — you'll create conflicts. Use the feature branch as the hand-off token: v0 pushes
   to it, then Claude Code pulls it (or vice-versa).
4. **The `specify` CLI is only needed to *init* a new project.** After init, the committed
   `.claude/` skills + `.specify/` templates run the whole workflow with no CLI — so the
   `/speckit-*` commands work in **both local and web** Claude Code identically. To start a
   brand-new project without the CLI (e.g. on web), just copy `.claude/skills` + `.specify` from
   the [SpecKit repo](https://github.com/MomentMan4/SpecKit) into the new repo and commit.
5. **Claude Code web setup (optional but handy).** The web environment clones the repo fresh and
   runs any configured setup script / `SessionStart` hook — use one to `pnpm install` (and install
   `specify-cli` if you ever init from web) so the session is ready to build and run tests.
   Outbound network on web follows the environment's network policy; if an install or fetch is
   blocked, that's the policy, not SDD.
6. **v0 is web-only and connects through GitHub.** Point v0 at the repo/feature branch; it pushes
   its work there, which is exactly the hand-off Claude Code expects. Nothing SDD-specific to
   configure in v0 beyond the GitHub connection.

Net: pick whichever surface is convenient for a given step — spec on web, implement locally, design
in v0, review on web — and let Git carry the state between them.

---

## 7. Notes & gotchas

- **Slash command names use hyphens** in this Spec-Kit version: `/speckit-specify`, not
  `/speckit.specify`.
- **Ephemeral environments:** in a fresh remote/cloud session the `specify` CLI won't persist,
  but the committed `.claude/` + `.specify/` files are enough to run the workflow. Re-install
  `specify-cli` only to `init` a brand-new project.
- **Keep `main` green**, work on feature branches; Vercel gives a preview per PR for visual
  review of v0-authored UI.

---

_Reference: <https://github.com/github/spec-kit> · Reusable setup: <https://github.com/MomentMan4/SpecKit>_
