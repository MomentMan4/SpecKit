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

## 5. How Claude + v0 fit together

- **v0 (Vercel)** authors and iterates the **UI** — generate components there, pull them into
  `components/`.
- **Claude Code + Spec-Kit** owns **engineering**: structure, logic, data flow, state,
  integrations, refactors — driven by the spec's tasks.
- In practice: define *what/how* in Spec-Kit → build UI in v0 → paste components in →
  `/speckit-implement` wires them to real data and the tasks (no leftover mock content).

The constitution already encodes this split, so every `/speckit-plan` respects it.

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
