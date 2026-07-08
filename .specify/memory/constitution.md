# MomentMan4 Project Constitution

<!--
Reusable baseline constitution for MomentMan4 projects.
Copy this file into any new Spec-Kit project (.specify/memory/constitution.md) and adjust
the [PROJECT-SPECIFIC] notes. It encodes the standard v0 + Claude Code + Next.js workflow so
every /speckit-plan and /speckit-implement run inherits these standards by default.

It covers the three kinds of projects built here: full applications, marketing websites, and
general/content websites. Principles apply to all three; where rigor should scale with the
project type, the "Right-Sized" guidance says so explicitly.
-->

## Project Types

Every project declares one of these types in its plan so the right level of rigor is applied.
The type sets expectations for testing, security, data, and state — a brochure site and a
multi-tenant app should not carry the same overhead.

- **Application** — software with real behavior: authentication, user/account data,
  persistence, APIs, non-trivial state, business logic (dashboards, SaaS, tools, portals).
  Highest rigor: security, data integrity, and automated testing are first-class.
- **Marketing website** — conversion-focused sites: landing pages, product/launch sites,
  waitlists. Content- and performance-led; light backend (forms, email capture, analytics).
- **Website (general/content)** — brochureware, portfolios, docs, blogs. Mostly static or
  CMS-driven content with minimal dynamic behavior.

A project may be a **pair** (e.g. a marketing site + its companion app) — treat each repo as
its own type.

## Core Principles

### I. Spec-Driven, Not Vibe-Driven
Every non-trivial feature starts as a written spec (`/speckit-specify`) before any code is
written. The spec describes **what** and **why** (user value, behavior, acceptance criteria)
and deliberately excludes implementation detail. Plans (`/speckit-plan`) own the **how**. Code
that has no corresponding spec/task is treated as unplanned scope and must be justified or
retro-specced. Ambiguity is resolved with `/speckit-clarify` before planning, never guessed
during implementation. This applies to all project types; applications additionally spec their
data model, auth, and key flows, not just screens.

### II. Design in v0, Engineer in Claude
UI is authored and iterated in **v0** (Vercel) and pulled into the repo; **Claude Code** owns
structure, logic, data flow, state, integration, and refactors. Neither hand-edits the other's
core concern without saying so: when Claude touches v0-generated components it preserves their
visual intent and Tailwind classes; when v0 output lands, Claude wires it to real data and the
spec's tasks rather than leaving mock content. Presentational components stay dumb; behavior
lives in hooks, server actions, services, and lib. For applications this separation is stricter:
UI never talks to the database or third-party APIs directly — it goes through a typed data/service layer.

### III. Standard Stack (NON-NEGOTIABLE unless the spec overrides)
The default stack is **Next.js (App Router) + TypeScript (strict) + Tailwind CSS + Radix/shadcn
primitives**, deployed to **Vercel**. For applications, the default backend is **Next.js server
actions / route handlers with a hosted Postgres (e.g. Supabase or Vercel Postgres) via a typed
query layer or ORM**, and authentication via a vetted provider (e.g. Supabase Auth, Auth.js) —
never hand-rolled crypto. New dependencies require a one-line justification in the plan. Prefer
platform and existing deps over new libraries (YAGNI). No `any` in committed TypeScript without
an inline reason. Server Components by default; `"use client"` only where interactivity demands it.

### IV. Data Integrity & Security (scales with project type)
Applies wherever a project stores data, authenticates users, or accepts input:
- **Validate at the boundary.** All external input (forms, params, webhooks, API bodies) is
  validated and typed at the server boundary (e.g. with a schema validator) before use.
- **Authorize every mutation and protected read.** Authentication is not authorization — every
  server action / route handler checks the caller is allowed to touch that specific resource.
  Prefer row-level security where the database supports it.
- **Least privilege & secrets.** Secrets live only in environment variables; service-role/admin
  keys never reach the client. Public (`NEXT_PUBLIC_*`) values are treated as public.
- **Safe data handling.** No secrets or PII in logs; parameterized queries only (no string-built
  SQL); explicit, documented data retention. Migrations are versioned and reviewed.

For marketing/content sites the practical floor is: validate form input, protect endpoints
against spam/abuse, and keep keys server-side. Full applications carry the whole list.

### V. Right-Sized Testing
Test depth scales with project type and risk, and the plan states the intended coverage:
- **Applications**: automated tests are required for business logic, data-layer/auth boundaries,
  and critical user flows (unit + integration; e2e for the highest-risk paths). Bugfixes add a
  regression test.
- **Marketing/content sites**: test custom logic (form handling, validation, dynamic pieces);
  visual/content correctness is verified on the Vercel preview.
Every change is verified end-to-end by exercising the actual behavior, not just by the build
passing.

### VI. Ship-Ready Increments
Every task closes in a state that builds and deploys clean: `pnpm build`, `pnpm lint`,
`pnpm typecheck`, and the project's test command (where tests exist) must pass before a task is
marked done. No committed secrets — all keys live in environment variables and `.env*` stays
gitignored; an up-to-date `.env.example` documents required variables. Broken `main` is never
acceptable; work happens on feature branches and merges only when green.

### VII. Accessible & Performant by Default
Semantic HTML, keyboard operability, focus management, and adequate color contrast are
acceptance criteria, not polish. Radix/shadcn primitives are used for interactive controls to
get a11y for free rather than reinventing them. Respect Core Web Vitals: optimize images via
`next/image`, avoid layout shift, keep client bundles lean, and lazy-load heavy/below-the-fold
work. Applications additionally guard perceived performance: loading/skeleton and error states
for async work, and no unbounded queries on hot paths.

### VIII. Compliance & Privacy by Design (Sense-and-Escalate)
No project proceeds past specification without a **compliance & privacy triage** (see the Risk,
Compliance & Governance section). The default posture is to actively *sense* whether the product
touches a regulated industry (healthcare, finance/payments, insurance, children, etc.) or
sensitive data (PHI, PII, cardholder data, biometric, credentials) and, if so, **escalate** to
the applicable privacy and regulatory controls rather than discovering them after launch. A
project that clears the triage records "no regulated industry / no sensitive data" explicitly —
silence is not a pass. Privacy and security controls are designed in from the first spec, never
retrofitted. Claude flags a suspected compliance obligation and pauses for human confirmation
instead of quietly building past it; this constitution is engineering guardrails, **not legal
advice**, and genuinely regulated products require qualified legal/compliance review.

## Technology & Structure Standards

- **Framework**: Next.js App Router. Routes/pages in `app/`, shared UI in `components/`,
  logic/helpers in `lib/`, reusable stateful logic in `hooks/`, server mutations in `actions/`.
- **Application data & backend** [PROJECT-SPECIFIC]: name the database, ORM/query layer, and
  auth provider in the plan. Keep data access behind a typed service/data layer in `lib/`
  (e.g. `lib/db`, `lib/services`) or server actions — never inline in components. Schema changes
  go through versioned migrations. External APIs are wrapped in a typed client, not called ad hoc.
- **State & data fetching**: Server Components + server actions by default; client state kept
  local and minimal; introduce a client data/query or global-state library only when the plan
  justifies it.
- **Styling**: Tailwind + `tailwind-merge`/`clsx` for conditional classes; design tokens in
  `tailwind.config.ts`; no ad-hoc inline styles when a utility class exists.
- **Components**: shadcn/Radix conventions; variants via `class-variance-authority`.
- **Errors & observability** (applications): typed error handling at boundaries, user-facing
  error/empty/loading states, and server-side error logging (no PII).
- **Environment**: all config via env vars; `.env.example` committed and current; secrets never
  committed.
- **Package manager**: pnpm (lockfile committed).
- **Deploy target**: Vercel (preview per PR, production on default branch).

## Risk, Compliance & Governance

This section is the "building it right" backbone. It runs on every project; its weight scales
with what the triage finds. For most marketing/content sites it is a two-minute check that comes
back clean. For anything touching regulated industries or sensitive data, it is mandatory and
gates launch.

### 1. Compliance & Privacy Triage (run during `/speckit-specify` and revisit at `/speckit-plan`)

Answer these sensing questions. **Any "yes" escalates** the project and pulls in the mapped
controls below. Record the answers and the resulting classification in the spec/plan.

- **Health**: Does it collect, store, or process health, medical, mental-health, or fitness
  data, or serve a healthcare provider/patient context? → **PHI**; regimes: **HIPAA** (US),
  **PHIPA/PIPEDA** (Canada), similar.
- **Payments/finance**: Does it take card payments, hold financial account data, or provide
  lending/investing/banking/insurance features? → **Cardholder data / financial data**; regimes:
  **PCI-DSS**, plus sector rules (e.g. SEC/FINRA, FCA, OSFI) for financial services.
- **Personal data**: Does it collect any personal data (name, email, IP, location, device IDs,
  behavioral/analytics profiles, user accounts)? → **PII**; regimes: **GDPR/UK GDPR** (EU/UK),
  **PIPEDA** and **Quebec Law 25** (Canada), **CCPA/CPRA** (California), and similar.
- **Children**: Could users be under 13 (or under 16 in the EU)? → **COPPA** / GDPR-K /
  age-assurance obligations.
- **Sensitive categories**: Biometric, precise geolocation, race/ethnicity, religion, sexual
  orientation, immigration, or union membership? → heightened "special category" protections.
- **Enterprise/trust**: Will customers require a security attestation? → **SOC 2 / ISO 27001**
  posture even absent a specific statute.
- **AI**: Does it make automated decisions about people or use AI on personal data? →
  transparency, human-review, and (EU AI Act) risk-tier considerations.

If every answer is "no," classify the project **Non-regulated / no sensitive data** and proceed
with the baseline guardrails only.

### 2. Data Classification

Label every data element the project handles; storage, access, and retention follow the label.

- **Public** — freely shareable (marketing copy, public content).
- **Internal** — non-sensitive operational data.
- **Personal (PII)** — identifies or relates to a person; privacy-by-design controls apply.
- **Sensitive** — PHI, cardholder/financial data, credentials/secrets, biometric, special
  categories; strongest controls, minimized, encrypted, tightly access-controlled, audited.

### 3. Privacy-by-Design controls (required when PII or PHI is present)

- **Data minimization & purpose limitation** — collect only what a stated purpose needs; don't
  repurpose silently.
- **Lawful basis & consent** — capture consent where required; make it specific and revocable;
  no pre-ticked boxes.
- **Data-subject rights** — support access, correction, export/portability, and deletion
  ("right to be forgotten") for personal data.
- **Retention & deletion** — documented retention schedule with automated or procedural deletion;
  no indefinite retention by default.
- **Security of processing** — encryption in transit (TLS) and at rest for sensitive data;
  least-privilege access; audit logging of access to sensitive records.
- **Third parties** — a Data Processing Agreement (DPA) with every sub-processor that touches
  personal data; maintain a list of sub-processors; verify their compliance posture.
- **Transparency & incident response** — a published privacy policy matching actual practice,
  and a breach-notification/incident-response plan with defined timelines.

### 4. Baseline Guardrails (never, on any project)

- No secrets, credentials, PII, or PHI in logs, error messages, analytics, client bundles, URLs,
  or the git repo.
- No production personal data in development, test, seed, or demo environments — use synthetic
  data.
- No sharing personal data with a third party (analytics, LLM APIs, email, etc.) without a DPA
  and a lawful basis; never send PHI/PII to a service not contracted for it.
- No hand-rolled cryptography or auth; use vetted providers.
- No storing raw card numbers — use a compliant processor (e.g. Stripe) and keep card data out
  of scope.
- No launching an escalated project without the sign-off in §6.

### 5. Risk Management (proportional to project type & triage)

Every plan includes a short **risk assessment**: the top risks (security, privacy, compliance,
data-loss, availability, reputational), each with likelihood × impact and a mitigation. High
likelihood-or-impact risks become explicit tasks with owners before launch. Revisit the
assessment when scope changes. Keep it lightweight for clean/non-regulated projects; make it
rigorous and documented for escalated ones.

### 6. Escalation & Sign-off

When the triage escalates a project:
- The applicable regime(s) and required controls are listed in the plan.
- A **compliance checklist** is generated (`/speckit-checklist`) and must be satisfied.
- Claude surfaces obligations and open legal questions and **does not treat them as resolved on
  its own** — a human owner confirms the approach, and (for genuinely regulated products) a
  qualified legal/compliance/security reviewer signs off **before production launch**.

## Development Workflow

1. `/speckit-constitution` — confirm/adjust these principles and declare the **project type**.
2. `/speckit-specify` — capture the feature's what/why + acceptance criteria (and data/auth
   shape for applications), and run the **compliance & privacy triage** (record the result).
3. `/speckit-clarify` (optional) — de-risk ambiguity before planning.
4. `/speckit-plan` — architecture and stack decisions consistent with this constitution;
   revisit the triage, record the data classification, and include the risk assessment.
5. `/speckit-tasks` — ordered, independently shippable tasks (including any risk/compliance
   mitigations as explicit tasks).
6. `/speckit-analyze` / `/speckit-checklist` (optional; **`/speckit-checklist` required for
   escalated projects**) — verify cross-artifact consistency and compliance-control coverage.
7. `/speckit-implement` — build against the tasks; keep build/lint/typecheck/tests green.

Quality gates before merge: build passes, lint clean, types check, required tests pass, spec
acceptance criteria met, input validated and mutations authorized (applications), no secrets
committed, a11y basics honored, and — for escalated projects — the compliance checklist
satisfied and privacy controls in place. UI changes are visually reviewed (Vercel preview)
before merge. Escalated projects also require the §6 sign-off before production launch.

## Governance

This constitution supersedes ad-hoc practice for projects that adopt it. Plans and
implementations that violate a principle must either be corrected or the principle amended
here first — deviations are documented in the plan's Complexity/Tradeoffs section, never left
implicit. The declared **project type** determines which "scales with project type" clauses
apply, but no project may silently skip a principle its type calls for. A project that the
triage escalates may not skip its compliance, privacy, or risk obligations under any deadline
pressure — descope the feature instead. Amendments bump the version below (semantic: MAJOR for
principle removals/redefinitions, MINOR for new principles/sections, PATCH for clarifications)
and update the Last Amended date. `CLAUDE.md` (if present) provides runtime guidance and must
stay consistent with this document.

**Version**: 1.2.0 | **Ratified**: 2026-07-08 | **Last Amended**: 2026-07-08
