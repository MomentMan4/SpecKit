# MomentMan4 Project Constitution

<!--
Reusable baseline constitution for MomentMan4 projects.
Copy this file into any new Spec-Kit project (.specify/memory/constitution.md) and adjust
the [PROJECT-SPECIFIC] notes. It encodes the standard v0 + Claude Code + Next.js workflow so
every /speckit-plan and /speckit-implement run inherits these standards by default.
-->

## Core Principles

### I. Spec-Driven, Not Vibe-Driven
Every non-trivial feature starts as a written spec (`/speckit-specify`) before any code is
written. The spec describes **what** and **why** (user value, behavior, acceptance criteria)
and deliberately excludes implementation detail. Plans (`/speckit-plan`) own the **how**. Code
that has no corresponding spec/task is treated as unplanned scope and must be justified or
retro-specced. Ambiguity is resolved with `/speckit-clarify` before planning, never guessed
during implementation.

### II. Design in v0, Engineer in Claude
UI is authored and iterated in **v0** (Vercel) and pulled into the repo; **Claude Code** owns
structure, logic, data flow, state, integration, and refactors. Neither hand-edits the other's
core concern without saying so: when Claude touches v0-generated components it preserves their
visual intent and Tailwind classes; when v0 output lands, Claude wires it to real data and the
spec's tasks rather than leaving mock content. Presentational components stay dumb; behavior
lives in hooks, server actions, and lib.

### III. Standard Stack (NON-NEGOTIABLE unless the spec overrides)
The default stack is **Next.js (App Router) + TypeScript (strict) + Tailwind CSS + Radix/shadcn
primitives**, deployed to **Vercel**. New dependencies require a one-line justification in the
plan. Prefer platform and existing deps over new libraries (YAGNI). No `any` in committed
TypeScript without an inline reason. Server Components by default; `"use client"` only where
interactivity demands it.

### IV. Ship-Ready Increments
Every task closes in a state that builds and deploys clean: `pnpm build`, `pnpm lint`, and
`pnpm typecheck` (or the project's equivalents) must pass before a task is marked done. No
committed secrets — all keys live in environment variables and `.env*` stays gitignored. Broken
`main` is never acceptable; work happens on feature branches and merges only when green.

### V. Accessible & Performant by Default
Semantic HTML, keyboard operability, and adequate color contrast are acceptance criteria, not
polish. Radix/shadcn primitives are used for interactive controls to get a11y for free rather
than reinventing them. Respect Core Web Vitals: optimize images via `next/image`, avoid
layout shift, keep client bundles lean, and lazy-load heavy/below-the-fold work.

## Technology & Structure Standards

- **Framework**: Next.js App Router. Routes/pages in `app/`, shared UI in `components/`,
  logic/helpers in `lib/`, reusable stateful logic in `hooks/`, server mutations in `actions/`.
- **Styling**: Tailwind + `tailwind-merge`/`clsx` for conditional classes; design tokens in
  `tailwind.config.ts`; no ad-hoc inline styles when a utility class exists.
- **Components**: shadcn/Radix conventions; variants via `class-variance-authority`.
- **Data & integrations** [PROJECT-SPECIFIC]: name the backend (e.g. Supabase, Vercel
  Postgres, external API) and auth approach in the project plan; keep data access in `lib/` or
  server actions, never inline in presentational components.
- **Package manager**: pnpm (lockfile committed).
- **Deploy target**: Vercel (preview per PR, production on default branch).

## Development Workflow

1. `/speckit-constitution` — confirm/adjust these principles for the project.
2. `/speckit-specify` — capture the feature's what/why + acceptance criteria.
3. `/speckit-clarify` (optional) — de-risk ambiguity before planning.
4. `/speckit-plan` — architecture and stack decisions consistent with this constitution.
5. `/speckit-tasks` — ordered, independently shippable tasks.
6. `/speckit-analyze` / `/speckit-checklist` (optional) — verify cross-artifact consistency.
7. `/speckit-implement` — build against the tasks; keep build/lint/typecheck green.

Quality gates before merge: build passes, lint clean, types check, spec acceptance criteria
met, no secrets committed, a11y basics honored. UI changes are visually reviewed (Vercel
preview) before merge.

## Governance

This constitution supersedes ad-hoc practice for projects that adopt it. Plans and
implementations that violate a principle must either be corrected or the principle amended
here first — deviations are documented in the plan's Complexity/Tradeoffs section, never left
implicit. Amendments bump the version below (semantic: MAJOR for principle
removals/redefinitions, MINOR for new principles/sections, PATCH for clarifications) and update
the Last Amended date. `CLAUDE.md` (if present) provides runtime guidance and must stay
consistent with this document.

**Version**: 1.0.0 | **Ratified**: 2026-07-08 | **Last Amended**: 2026-07-08
