# SpecKit — a plain-English guide

This is how we build software here: you **write down what you want first** (a spec), let the tools
turn that into a plan and tasks, then build against it. It's called **Spec-Driven Development
(SDD)**. This guide assumes no prior experience — if a word is unfamiliar, check the
[glossary](#glossary) near the bottom.

You build with two tools, and they share one project through **GitHub**:

- **Claude Code** — an AI coding assistant. You run it on your computer (the **desktop app** or
  the terminal) and on the **web** at claude.ai/code. It runs the SpecKit commands and writes code.
- **v0** (v0.dev, by Vercel) — an AI builder you use in the browser. It builds UI *and* backend and
  pushes its work to GitHub.

You can build any feature with **either** tool — you choose per feature. SpecKit keeps them working
from the same plan.

---

## The big picture (read this once)

1. **Write a spec** — describe *what* you want and *why* (not how). → `/speckit-specify`
2. **Get a plan** — the tools decide *how* to build it. → `/speckit-plan`
3. **Break it into tasks** — a to-do list. → `/speckit-tasks`
4. **Build it** — in Claude Code and/or v0.
5. **Check and ship it** — review, test, deploy.

Everything you produce (the spec, the plan, the code) is saved to **GitHub**, so whichever tool or
device you pick up next sees the same thing.

> **Golden rule:** the **spec** says *what & why*; the **plan** says *how*. Don't put code details
> in the spec, and don't invent new scope while building — update the spec instead.

---

## What you need

| You need | Why | Notes |
|---|---|---|
| A GitHub account + a repo | Stores the project; syncs the tools | The place everything lives |
| Claude Code (desktop, terminal, or web) | Runs SpecKit + writes code | You likely already have this |
| A v0 (Vercel) account | Optional — for building in v0 | Only if you use v0 |
| Git installed (for local work) | Save/sync your work | Comes with the desktop dev tools |
| Python 3.11+ and `uv` | Only to *create* SpecKit in a new repo | Skip if you copy the files instead (below) |

---

## Set up

There are two layers: **(A) put SpecKit into your project once**, then **(B) open that project on
whatever you're using** — Claude Code desktop, Claude Code web, or v0.

### A. Put SpecKit into your project (once per repo)

SpecKit is just a set of files in your repo: a `.claude/` folder (the commands) and a `.specify/`
folder (the templates + your rulebook, called the *constitution*). Add them one of two ways:

**Easiest — copy them from the SpecKit repo (no extra tools):**
```bash
git clone --depth 1 https://github.com/MomentMan4/SpecKit /tmp/speckit
mkdir -p ./.claude
cp -r /tmp/speckit/.claude/skills ./.claude/skills
cp -r /tmp/speckit/.specify ./.specify
git add .claude .specify && git commit -m "Add SpecKit" && git push
```

**Or — generate fresh with the CLI (needs Python 3.11+ and `uv`):**
```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
cd <your-project>
specify init --here --integration claude --force
```

**Starting a brand-new project?** Use SpecKit itself as the seed:
```bash
git clone https://github.com/MomentMan4/SpecKit my-app
cd my-app && rm -rf .git && git init      # start a clean history for your app
```

After adding the files, open `.specify/memory/constitution.md` and adjust the two
`[PROJECT-SPECIFIC]` spots (your database/auth and data layer). **Commit and push** so every tool
sees it. Never commit secrets — keep them in `.env` files (already ignored by Git).

### B. Open the project on your surface

**Claude Code — desktop app or terminal (your computer):**
1. Make sure the project folder on your computer has the `.claude/skills` folder in it.
2. Open the project **at its top level** — desktop: *Open Project* and pick the folder that
   contains `.claude/`; terminal: `cd` into it and run `claude`.
3. Type `/` — you should see the `speckit-` commands. Start with `/speckit-constitution`.
4. *Don't see them?* Check that `.claude/skills/speckit-constitution/SKILL.md` sits at the top of
   the project (not in a subfolder), then reopen the project so it re-scans.

**Claude Code — web (claude.ai/code):**
1. Start a session on your **GitHub repo and branch**. It downloads a fresh copy automatically, so
   the commands are already there — nothing to install.
2. Work as normal, but **commit and push before you leave** (see Git basics) — the web workspace is
   temporary and forgets anything you didn't push.

**v0 (browser):**
1. Sign in at **v0.dev** with your Vercel account.
2. **Connect GitHub** and point v0 at your **repo and branch**. That's the whole setup — v0 saves
   its work straight to that branch, which Claude Code can then pick up.
3. Paste your design rules (from the constitution) and the relevant user stories into v0 so its
   output matches your standards.

---

## The SpecKit commands

Type these in Claude Code (they start with `/`). Run them in order.

| Command | Plain-English job |
|---|---|
| `/speckit-constitution` | Set or review your project's rules and standards. Run once at the start. |
| `/speckit-specify` | Describe the feature: what it does and why. **No tech details.** |
| `/speckit-clarify` *(optional)* | Lets Claude ask you questions to remove any guesswork before planning. |
| `/speckit-plan` | Turns the spec into a technical plan: architecture, stack, decisions. |
| `/speckit-tasks` | Breaks the plan into an ordered to-do list. |
| `/speckit-analyze` *(optional)* | Checks the spec, plan, and tasks agree with each other. |
| `/speckit-checklist` *(optional)* | Makes a quality checklist (required for regulated projects). |
| `/speckit-implement` | Builds the feature by working through the tasks. |
| `/speckit-converge` | Looks at existing code and adds any missing work as new tasks. |
| `/speckit-taskstoissues` | Turns the tasks into GitHub issues. |

---

## Your first feature, step by step

Say you want to add an email waitlist to a landing page.

**1. Set the rules (once per project).** In Claude Code:
```
/speckit-constitution
```
Review the standards and tell it your **project type** (Application, Marketing website, or
Website).

**2. Describe what you want.**
```
/speckit-specify
```
In plain words: *"Add an email waitlist. Visitors enter their email in the hero and near the
footer, see a success message, and we prevent duplicate signups."* Answer the privacy questions it
asks.

**3. (Optional) Let it clarify, then plan.**
```
/speckit-clarify        (optional)
/speckit-plan
```
Now you have `spec.md`, `plan.md` under `specs/…` in your repo.

**4. Make the task list.**
```
/speckit-tasks
```

**5. Decide who builds it — v0 or Claude Code** (you can split, too):
- **v0** — great for building screens fast and simple backends you want to see running quickly.
- **Claude Code** — great for complex logic, big changes, tests, and tricky integrations.

**6. Build it.**
- In Claude Code: `/speckit-implement`
- In v0: prompt it with the spec + your design rules, then push to your branch.

**7. Check and ship.**
- `/verify` — actually click through the feature, don't just trust that it built.
- `/code-review` (and `/security-review` if it touches data, logins, or payments).
- Open a **Pull Request** on GitHub → automatic checks run and Vercel makes a **preview link** →
  review it → **merge** → it goes live.

**8. Next feature:** start again at `/speckit-specify`.

---

## Using SpecKit in a project that already has code

You don't need a blank project. Adding SpecKit to an existing app is the same setup, with a few
adjustments:

1. **Add the files on a branch.** Do the copy step from [Set up A](#a-put-speckit-into-your-project-once-per-repo)
   on a new branch (e.g. `chore/add-speckit`), then open a PR and merge. If the project already has
   a `.claude/` folder, **merge** the `speckit-` skills into it — don't replace the whole folder.
2. **Make the rulebook match what you already use.** Run `/speckit-constitution`, set the
   **project type**, and adjust the standards + the `[PROJECT-SPECIFIC]` spots so they reflect your
   real stack (framework, database, auth). If the repo has a `CLAUDE.md`, keep it consistent with
   the constitution.
3. **Adopt it going forward — you do NOT re-spec the whole app.** From here:
   - **New feature or change** → the normal flow: `/speckit-specify` → `/speckit-plan` →
     `/speckit-tasks` → build, on a branch.
   - **Want SpecKit to account for what's already built?** Run **`/speckit-converge`** — it looks at
     your existing code against a spec/plan and adds the *remaining* work as tasks. This is the
     command made for existing projects.
   - *(Optional)* Reverse-engineer a short spec for one important area if you want a written record
     of how it works today — but don't try to document everything at once.
4. **Improve old code as you pass through it.** When a new specced feature touches older code, bring
   that part up to the constitution's standards then — steady cleanup, not one big rewrite.

> First time on an existing repo? Add the files, run `/speckit-constitution` to fit your stack, then
> use `/speckit-specify` for your **next** feature. That's enough to start.

---

## Git basics — save and sync your work

**Git** is the system that saves your work and shares it between tools. **GitHub** is the website
that hosts it. You mostly do four things. (In Claude Code you can also just *ask in plain English* —
"pull the latest," "commit and push" — and it runs these for you.)

| Action | What it means | Command |
|---|---|---|
| **Pull** | Download the latest work before you start | `git pull` |
| **Commit** | Save a snapshot of your changes, with a message | `git add -A && git commit -m "what I did"` |
| **Push** | Upload your saved snapshots to GitHub | `git push` |
| **Status** | See your current branch and what changed | `git status` |

**Pull before you start:**
```bash
git checkout <your-branch>      # move onto your branch
git pull                        # get the latest
```

**Push when you're done:**
```bash
git add -A                      # stage all your changes
git commit -m "add waitlist form"
git push                        # upload
```
First time pushing a new branch? Use `git push -u origin <your-branch>` once; after that plain
`git push` works.

> If a push is rejected saying you're "behind," someone else pushed while you worked. Run
> `git pull` (fix any conflict it points out), then `git push` again.

---

## Branches — always work on one (not `main`)

A **branch** is a safe, separate copy of the project to work on. `main` is the live version that
gets deployed — you never build directly on it. **Make a new branch for each feature.**

Why it's worth it:
- `main` stays safe and deployable while you experiment.
- Your branch gets a **Pull Request**, automatic checks, and a **Vercel preview link** to review
  before anything goes live.
- It's how v0 and Claude Code hand work back and forth — one branch, one feature, one tool at a
  time.
- A bad branch is easy to throw away; a broken `main` is a headache.

**The full branch loop:**
```bash
git checkout main && git pull                 # start from the latest main
git checkout -b feature/waitlist              # create + move onto a new branch
# ...build, then save and upload:
git add -A && git commit -m "add waitlist"
git push -u origin feature/waitlist           # first push of a new branch
# → open a Pull Request on GitHub, review the preview, merge it, then:
git checkout main && git pull                  # bring the merged work back
git branch -d feature/waitlist                 # delete the finished branch
```
Name branches clearly: `feature/…`, `fix/…`, `chore/…`.

**In Claude Code** you can just say: *"Create a branch called feature/waitlist and work there."*
**In v0**, push its output to that same branch.

The only time you can skip a branch is a throwaway experiment you'll never keep.

---

## Using more than one surface (desktop + web + v0)

SDD works the same everywhere because everything lives in GitHub. The tools **only** share work
through it, so treat GitHub as the single meeting point:

1. **Push everything that matters** — especially from Claude Code web, which forgets unpushed work.
2. **Pull before you start** on any surface; **push when you finish.**
3. **One surface per branch at a time.** Don't edit the same branch in v0 and Claude Code at once,
   or they'll collide. Hand work off through the branch: v0 pushes → Claude Code pulls, or the
   reverse.
4. **Commit the spec files too** (`.specify/`, `specs/`), not just code — that's how the next tool
   gets the plan and tasks.

---

## FAQ & common gotchas

**Do I have to use both v0 and Claude Code?** No. Use whichever fits. Many features are built
entirely in one.

**The `/speckit-` commands don't show up.** You're probably not opened at the project's top level,
or `.claude/skills/` isn't there/committed. Open the folder that contains `.claude/` and reopen the
project.

**The command is `/speckit-specify`, with a hyphen** — not `/speckit.specify`.

**Do I need the `specify` CLI every time?** No. It's only for *creating* SpecKit in a new repo.
Once the `.claude/` and `.specify/` files are committed, the commands work everywhere without it.

**I lost work in a Claude Code web session.** The web workspace is temporary. Always commit and
push before you finish. (Local/desktop work stays on your computer, but still push it.)

**Something's broken on the live site.** You likely merged to `main` before it was ready. Work on a
branch and use the preview link next time; for now, revert the bad change on `main`.

**Where do secrets (API keys) go?** In `.env` files, which Git ignores. Never commit them.

---

## Glossary

- **SDD (Spec-Driven Development)** — write the spec first; it drives the build.
- **Spec** — a plain-English description of *what* a feature does and *why* (`spec.md`).
- **Plan** — the technical *how* for a spec (`plan.md`).
- **Tasks** — the ordered to-do list built from the plan (`tasks.md`).
- **Constitution** — your project's rulebook (`.specify/memory/constitution.md`): standards for
  stack, security, testing, design, AI, and compliance. The tools follow it.
- **Repo (repository)** — your project's folder, tracked by Git and hosted on GitHub.
- **Branch** — a separate copy of the project to work on safely; `main` is the live one.
- **Commit** — a saved snapshot of changes, with a message.
- **Push / Pull** — upload your commits to GitHub / download the latest from GitHub.
- **Pull Request (PR)** — a request to merge your branch into `main`; where review and checks happen.
- **CI** — automatic checks (build, lint, tests) that run on your PR.
- **Preview** — a temporary live version of your branch that Vercel builds so you can see it before
  merging.
- **Merge** — combining your finished branch into `main` (which then deploys).

---

_Reference: <https://github.com/github/spec-kit> · Reusable setup & rulebook:
<https://github.com/MomentMan4/SpecKit>_
