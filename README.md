# SpecKit — Spec-Driven Development starter (Claude + v0)

A reusable **Spec-Driven Development (SDD)** setup for MomentMan4 projects, built on
[GitHub Spec-Kit](https://github.com/github/spec-kit). Copy it into any project to get the
`/speckit-*` workflow (as Claude Code slash commands) plus a shared **constitution** that
encodes the standard **v0 + Claude Code + Next.js / TypeScript / Tailwind / Radix / Vercel**
way of working.

With SDD, the **spec is the source of truth**: you write a spec → generate a plan → break it
into tasks → let the agent implement against them, instead of prompting ad-hoc.

---

## What's in this repo

```
.claude/skills/speckit-*/SKILL.md   # the /speckit-* Claude Code slash commands
.specify/
  memory/constitution.md            # reusable MomentMan4 constitution (project principles)
  templates/                        # spec, plan, tasks, checklist, constitution templates
  scripts/                          # helper scripts the commands call
  workflows/                        # workflow registry
```

The `constitution.md` is the reusable core. Its five principles:

1. **Spec-driven, not vibe-driven** — every feature starts as a written spec.
2. **Choose the implementer** — v0 or Claude Code (both are full-stack builders); declare it per feature.
3. **Standard stack** — Next.js App Router + TS (strict) + Tailwind + Radix/shadcn → Vercel.
4. **Ship-ready increments** — build/lint/typecheck green, no committed secrets.
5. **Accessible & performant by default** — a11y and Core Web Vitals are acceptance criteria.

---

## Requirements

| Requirement | Why |
|---|---|
| Python 3.11+ | Runs the `specify` CLI |
| uv (or pipx) | Installs `specify-cli` |
| Git | Spec-Kit is git-aware (a feature branch per spec) |
| Claude Code | Runs the `/speckit-*` slash commands |
| v0 account (Vercel) | optional — UI generation |

You only install `specify-cli` **once per machine**. After a project is initialized, the CLI is
not needed again — the committed `.claude/` skills and `.specify/` templates run the whole
workflow.

---

## Use it in a new project

**Option A — `specify init` (recommended, always current):**

```bash
# once per machine
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# per new project
cd <your-project>
specify init --here --integration claude --force
# add --ignore-agent-tools if the `claude` binary isn't on PATH

# then seed your standards:
# copy this repo's .specify/memory/constitution.md into the new project and
# adjust the two [PROJECT-SPECIFIC] spots (backend/auth, data layer)
```

**Option B — copy from this repo (fast, pins these exact versions):**

```bash
# from your project root
git clone --depth 1 https://github.com/MomentMan4/SpecKit /tmp/speckit
cp -r /tmp/speckit/.claude/skills /your-project/.claude/skills   # merge, don't clobber other skills
cp -r /tmp/speckit/.specify /your-project/.specify
```

Either way, commit `.claude/` and `.specify/` to the project. Secrets stay in `.env*`
(already gitignored) — never commit keys.

---

## The workflow (Claude Code slash commands)

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

---

## How Claude + v0 fit together

- **Spec-Kit (in Claude Code)** owns the *thinking* for every project — spec, plan, tasks.
- **Implementation is your choice per feature.** Both **v0** and **Claude Code** are capable
  full-stack builders — v0 does UI **and** backend (routes, server actions, data wiring), not just
  visuals. Assign each feature to v0, to Claude Code, or to a split, by fit.
- **Whichever tool builds, the constitution's standards apply** (typed data layer, design tokens,
  all states, validation/auth, tests) so the *other* tool can extend it without a rewrite — a
  clean hand-off in both directions.

The constitution encodes this (Principle II — "Choose the Implementer"), so every `/speckit-plan`
records who builds each feature.

👉 **For the full, numbered step-by-step** (Phase 0 setup → spec/plan in Claude → choose the
implementer → build in v0 and/or Claude Code → QA/review/ship), see **[`SPEC-KIT.md`](./SPEC-KIT.md)
§ "Using SpecKit with Claude and v0 — step by step".**

---

## Notes & gotchas

- **Slash command names use hyphens** in this Spec-Kit version: `/speckit-specify`, not
  `/speckit.specify`.
- **Ephemeral environments:** in a fresh remote/cloud session the `specify` CLI won't persist,
  but the committed `.claude/` + `.specify/` files are enough to run the workflow. Re-install
  `specify-cli` only to `init` a brand-new project.
- **Keep `main` green**, work on feature branches; Vercel gives a preview per PR for visual
  review of v0-authored UI.
- **Using Claude Code locally *and* on the web, plus v0 on the web?** SDD works the same on all
  three — the spec, skills, and code live in Git, which is the sync point. Commit + push everything
  (the web environment is ephemeral), pull before switching surfaces, and keep one surface per
  branch at a time. See [`SPEC-KIT.md`](./SPEC-KIT.md) § "Working across surfaces".

---

_Based on [github/spec-kit](https://github.com/github/spec-kit). Generated with Claude Code._
