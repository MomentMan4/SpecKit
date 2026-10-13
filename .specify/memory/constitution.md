<!--
Sync Impact Report
- Version change: 1.6.0 → 2.0.0 (MAJOR: Principle V redefined with enforceable MUST/SHOULD rules)
- Modified principles:
  - V. Testing & Quality Assurance (Right-Sized) → V. Testing & Quality Assurance
    (Evidence-Based, Right-Sized) — full replacement: doctrines, hierarchy, Playwright default,
    critical journeys, AC evidence, resilient tests, auth/env, contracts, regression/flake policy,
    failure evidence, browser matrix, a11y/visual, CI/release gates, Definition of Done
  - I. Spec-Driven — adds AC IDs, verification methods, and state listing
  - IV. Data Integrity & Security — adds "Prove it" (negative-path + migration testing)
  - IX. AI Capabilities — adds software tests vs evals split
- Added sections: Verification Standards (§1–§8: traceability, security testing, state coverage,
  API/contract testing, database change safety, observability, AI tests vs evals, evidence
  record), Supporting Standards, Exceptions; Testing entry in Technology & Structure Standards;
  Delivery flow in Development Workflow
- Removed sections: none
- Templates:
  ✅ .specify/templates/spec-template.md (AC IDs + methods, applicable states)
  ✅ .specify/templates/plan-template.md (Constitution Check gates, Verification Plan)
  ✅ .specify/templates/tasks-template.md (verification paired with implementation)
  ✅ .claude/skills/speckit-tasks/SKILL.md (tests no longer optional)
  ✅ .specify/standards/*.md created (testing, playwright, security-testing, data-migrations,
     ai-evaluation)
- Follow-up TODOs: none
-->
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
data model, auth, and key flows, not just screens. Every behavioral acceptance criterion carries
a stable ID and a verification method, and the spec names the materially reachable states, so
verification is traceable from spec to plan to tasks to evidence (Principle V; Verification
Standards §1 and §3).

### II. Choose the Implementer — v0 or Claude Code (declare it per feature)
Both **v0** and **Claude Code** are capable builders — v0 generates full-stack Next.js (UI **and**
backend: route handlers, server actions, data wiring), and Claude Code implements across the whole
stack too. There is **no fixed "v0 does UI, Claude does logic" rule**. The plan **declares who
implements each feature or task** — v0, Claude Code, or a split — chosen by fit (see "Choosing the
Implementer" in the workflow), and the choice can differ feature to feature and change over a
project's life. **Whichever tool builds, the output must satisfy this constitution** — typed
data/service layer, design tokens, all states, validation and authorization, tests — so the *other*
tool can pick it up and extend it without a rewrite. Hand-offs are clean in both directions: real
data wired (no leftover mock content), behavior in hooks/actions/services/`lib`, presentational
components kept dumb, and — for applications — UI never talks to the database or third-party APIs
directly regardless of which tool wrote it. When one tool edits the other's work, it preserves
intent (visual design, Tailwind classes, and existing structure) rather than rewriting for style.

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
- **Prove it.** Authorization, tenant isolation, and input handling are verified by negative-path
  tests at the server boundary, and migrations are tested against existing data with a recovery
  strategy (Verification Standards §2 and §5).

For marketing/content sites the practical floor is: validate form input, protect endpoints
against spam/abuse, and keep keys server-side. Full applications carry the whole list.

### V. Testing & Quality Assurance (Evidence-Based, Right-Sized)
Testing MUST demonstrate that the implemented system satisfies its specification. A passing
build is necessary but insufficient. Test depth MUST scale with project type and risk, and each
test MUST prove a meaningful behavior. The detailed standards in `.specify/standards/` (see
Supporting Standards) govern implementation conventions.

**Key words.** MUST states a requirement; SHOULD permits an exception only when the plan
justifies it; MAY is optional.

**Testing doctrines.**
1. **Evidence proves behavior.** A passing build alone does not prove a feature works.
2. **Test as low as possible, as high as necessary.** Match the test layer to the risk and the
   behavior.
3. **Specifications own verification.** Every behavioral acceptance criterion has a method and
   evidence.
4. **Validate real boundaries.** Do not mock away the boundary a test claims to prove.
5. **Determinism and isolation are requirements.** Flaky tests are defects.
6. **Failures and states matter.** Prove denials, recovery, and data integrity as well as
   success.
7. **Completion includes verification.** Regression evidence, CI gates, and review come before
   shipping.

#### Testing hierarchy
Use the lowest-cost layer that reliably proves the behavior. Do not duplicate exhaustive
lower-level cases in E2E, and do not mock away the system boundary under test.
- **UNIT** — business rules, calculations, transformations, validators, policy decisions, and
  deterministic utilities.
- **INTEGRATION** — data access, database invariants, APIs, server actions, authorization
  boundaries, webhooks, and provider adapters. Component tests MAY verify complex interactions
  where useful.
- **E2E** — Playwright user journeys spanning browser behavior, navigation, authentication,
  persistence, and other system boundaries.
- **MANUAL** — explicit human verification when automation is impractical, including complex
  accessibility review.
- **EVAL** — measured AI behavior against a versioned evaluation set (see AI Engineering
  Standards §5).

#### Default framework and critical journeys
Browser applications MUST use **Playwright** for E2E testing unless the plan records a justified
alternative. Non-browser products MUST select an appropriate system-testing tool. Critical
journeys MUST have E2E coverage. A journey is **critical** when its failure blocks product
access or primary value, corrupts important data, violates privacy or authorization, affects
billing or account state, or causes irreversible or materially harmful actions. Cover the
applicable login, onboarding, primary CRUD, protected actions, checkout, uploads, and critical
AI workflows. Real integration tests MUST prove server-side and tenant boundaries even when UI
coverage is present.

#### Acceptance criteria and evidence
Every behavioral acceptance criterion MUST have a stable ID (`AC-01`, `AC-02`, …) and one or
more verification methods: `[UNIT]`, `[INTEGRATION]`, `[E2E]`, `[MANUAL]`, or `[EVAL]`. The plan
MUST identify the test layer, applicable states, environment, and evidence required. Tasks MUST
include verification work together with the implementation. Completion MUST link each criterion
to passing test results or recorded manual/eval evidence. **Unmapped criteria are incomplete.**
See Verification Standards §1 for the traceability format.

#### Resilient browser tests
Tests MUST assert user-visible outcomes using semantic locators and web-first assertions. Prefer
role and accessible name, labels, and meaningful visible text before test IDs. CSS, XPath,
generated classes, DOM hierarchy, and positional selectors require an unavoidable, documented
reason.

Tests MUST use locator auto-waiting, retrying assertions, or explicit observable events.
Arbitrary `waitForTimeout()` calls MUST NOT be committed as a synchronization strategy;
temporary diagnostic sleeps MUST be removed before merge. Tests MUST run independently and in
any order, own their data, and control clocks, randomness, and fixtures where these affect
results.

#### Authentication and environment
Authenticated suites SHOULD use setup projects and reusable `storageState` where appropriate.
Saved state MUST be stored in a gitignored path such as `playwright/.auth/`, contain only
dedicated non-production accounts, and never be committed or exposed as a public artifact.
Generate fresh state per run as needed; explicitly test login, logout, expiry, and revocation
rather than bypassing them with saved state.

Mutating tests MUST isolate accounts and server-side data across workers, shards, and retries —
browser-context isolation alone does not isolate the database. Fixtures MUST create known users,
roles, and records, use unique run identifiers, and clean up safely or use disposable
environments. Time-dependent assertions MUST use controlled clocks and explicit time zones.

Tests MUST NOT depend on production customer data. Full-system validation SHOULD use a
deployable preview or staging environment (e.g. the Vercel preview) with production-like
configuration. Seed/reset mechanisms MUST be inaccessible in production. Production smoke checks
MAY exercise approved, safe, read-only paths; any customer-visible mutation requires an
explicitly designed and authorized test mechanism.

#### External systems and contracts
Each external integration MUST define its verification strategy. Use deterministic stubs for
ordinary tests involving costly, destructive, rate-limited, or nondeterministic providers; stubs
MUST match versioned provider contracts. Important integrations MUST also have appropriate real
adapter or sandbox verification — mocked success alone is insufficient. Payments MUST use
provider test modes. Live provider checks SHOULD run separately from stable PR gates when
availability or cost makes them unsuitable for every change.

#### Regression and flake policy
Every confirmed defect MUST receive a regression test at the lowest appropriate layer before or
alongside the fix. If automation is genuinely impractical, record reproducible manual evidence
and an approved exception. Verify that the regression test reproduces the original defect when
feasible.

Flaky tests MUST be investigated as defects. CI MAY retry a limited number of times for
diagnosis, but a retry-pass MUST remain visible as flaky and MUST NOT automatically satisfy the
quality gate. Quarantine requires an owner, issue, reason, expiry, and compensating coverage;
critical journeys MUST NOT silently lose verification. Increasing waits, retries, or timeouts
without addressing the cause is not a fix.

#### Failure evidence
CI MUST retain a test report and useful failure diagnostics: traces, failure screenshots, and
relevant browser, console, and server errors. Add video only where useful. Local retries SHOULD
be 0; CI retries SHOULD be limited to 1 or 2. Use retain-on-failure traces (or on-first-retry
with retries enabled), failure-only screenshots, and HTML/JUnit reports. Artifact access and
retention MUST protect tokens, personal data, and request bodies.

#### Browser and viewport coverage
The plan MUST name the materially supported browsers and viewports. Chromium SHOULD be the
default PR browser. Mobile-critical products MUST include representative mobile viewport/device
coverage. Firefox and WebKit MUST be included when support commitments or risk require them;
wider matrices MAY run on main, nightly, or release candidates. Emulation does not replace
real-device checks for platform-specific risk.

#### Accessibility and selective visual regression
Critical journeys MUST verify relevant keyboard operation and focus behavior. Major pages and
components SHOULD include automated accessibility checks. Teams MUST also verify accessible
names, semantics, contrast, and complex interactions through appropriate manual review — an
automated scan is not complete accessibility evidence (Principle VII).

Visual snapshots SHOULD target stable, high-value surfaces such as design-system components,
important dashboards, and regression-prone responsive layouts. Control fonts, animations,
timestamps, data, operating system, and browser versions. Baseline changes MUST be reviewed
against intended design changes, never accepted automatically to clear failures.

#### Right-sized expectations
- **Applications**: meaningful unit tests; data, auth, and API integration tests; Playwright
  critical journeys; negative paths and regression coverage.
- **Marketing websites**: navigation and conversion smoke tests, form validation/submission,
  responsive behavior, accessibility, and important analytics events where practical.
- **Content websites**: critical rendering/navigation smoke checks, applicable link/form checks,
  responsive behavior, and accessibility. Extensive suites require corresponding product risk.

#### CI and release gates
Pull requests MUST run the applicable lint, typecheck, unit tests, integration tests, build, and
the application's Playwright critical-path suite. Risk-based selection MUST include changed
boundaries and dependent journeys. Broader suites MAY run on main, nightly, or release
candidates, but required release coverage MUST pass before shipping — a scheduled job does not
substitute for a required pre-release check.

Tests MUST target the candidate build in an isolated environment. CI MUST fail on focused tests
(`.only`), required-test failures, missing required execution, and unresolved flaky results.
Test reports and failures MUST remain visible. Changes to models, prompts, retrieval, or tool
policy MUST receive relevant evaluation evidence before release. Every change MUST be
code-reviewed before merge — human review and/or `/code-review` + `/security-review`; no direct
commits to `main`.

#### Definition of Done
A task/feature is complete only when: the specification is satisfied; **every acceptance
criterion has evidence**; required tests and CI checks pass; critical journeys work against a
deployable candidate; bug fixes include regression verification; applicable security, state,
API, migration, accessibility, and responsive checks pass; and no unexplained browser, console,
or server errors remain. Documentation and `.env.example` MUST be current. Review MUST confirm
operational readiness, relevant evaluation results, and any explicit exceptions — and, for
escalated projects, that the required compliance controls are in place. The full checklist lives
in `.specify/standards/testing.md`.

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

### IX. AI Capabilities — Spec the Model *and* the Behavior
When a feature uses AI (an assistant, agent, RAG/search, classifier, generator, LLM-judge,
extraction, summarization, etc.), the spec/plan must **name the capability precisely and pin
the model** — vague "add AI" is not a spec. Every AI capability declares: (a) the **use case and
industry** it serves and the **scope of what it may and may not do** (stated as capabilities and
explicit non-goals, correct for that domain — a healthcare triage assistant and a marketing copy
generator have very different allowed behaviors); (b) the **Claude model + reasoning effort**
(see AI Engineering Standards); (c) its **tools, data access, and grounding sources**; (d) its
**guardrails** — refusal/safety handling, escalation-to-human path, and what it must never claim
or do; (e) how it is **evaluated** (a small eval set / rubric, not vibes). AI on personal data or
in a regulated domain inherits Principle IV and VIII in full: disclose AI use, keep a human in
the loop for consequential decisions, and never send PHI/PII to a model, provider, or gateway not
contracted for it. Model choice is not locked to one vendor — Claude is the house default, but
other reputable providers (and Vercel AI Gateway for routing across them) are allowed when they
fit; pick and pin the exact model per the AI Engineering Standards, and confirm the model and API
surface against the provider's current docs before shipping rather than hardcoding IDs "from
memory". AI features carry both deterministic software tests and model evaluations — neither
substitutes for the other (Verification Standards §7).

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
- **Monitoring in production** (applications): error tracking + alerting (e.g. Sentry), uptime,
  and performance monitoring (Vercel Analytics / Speed Insights) so failures are seen by the team,
  not discovered by users. Wire this before launch, not after the first incident.
- **SEO & metadata** (marketing/content sites and any public app page): use the Next.js Metadata
  API for per-route `title`/`description`; Open Graph + Twitter cards; `sitemap.ts` and
  `robots.ts`; canonical URLs; one semantic `h1` per page and descriptive image `alt` text; add
  JSON-LD structured data where it helps. For marketing sites, SEO basics are acceptance criteria.
- **Analytics & consent** (marketing/content): measure the key conversions the spec cares about
  with a privacy-respecting analytics setup; gate non-essential tracking behind consent where the
  triage requires it (Principles IV & VIII).
- **CI/CD**: quality gates (build, lint, typecheck, unit/integration tests, and the Playwright
  critical-path suite against the candidate) run automatically on every PR (e.g. GitHub Actions);
  Vercel gives a preview per PR and ships production on the default branch. See Principle V and
  `.specify/standards/testing.md` for the pipeline sequence.
- **Testing** [PROJECT-SPECIFIC]: name the unit/integration runner (e.g. Vitest) in the plan;
  browser E2E uses Playwright (`playwright.config.ts`, tests in `tests/e2e/`, auth state in a
  gitignored `playwright/.auth/`). See `.specify/standards/playwright.md`.
- **Dependencies**: pnpm lockfile committed; keep the dependency surface small (YAGNI); watch for
  and patch known vulnerabilities; pin and vet anything security-sensitive.
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

## AI Engineering Standards

How AI capabilities are built so both implementers — Claude Code and v0 — produce them well and
consistently. This section is authoritative for any feature that calls an LLM, whichever tool
builds it.

### 1. Choose the right model & provider (not locked to one vendor; pin the exact model)

Model choice is **not restricted to a single vendor.** Pick the model that best fits the task on
capability, cost, latency, context window, modality, and — critically — **data-residency /
compliance fit**, and **pin the exact model ID and provider** in the plan (no vague "an LLM", no
invented date-suffixed IDs).

- **House default: Anthropic Claude**, but **OpenAI, Google Gemini, and other reputable providers
  are allowed** when they fit the task better or the plan justifies them. Pick the equivalent tier
  (frontier / balanced / fast-cheap) by task, not by habit.
- **Integration layer — the Vercel AI SDK (`ai`)** is the recommended way to call models on this
  Next.js/Vercel stack: one provider-neutral interface (`streamText` / `generateText` /
  `generateObject`) so swapping models/providers is a config change, not a rewrite. Use a
  provider's official SDK directly only when you need a capability the AI SDK doesn't expose.
- **Vercel AI Gateway is an approved option** — route model calls through one endpoint to
  switch/fail over across providers and models, centralize keys and budgets, and get unified
  observability without managing each provider key in the app. Prefer it when using multiple
  providers or when you want provider failover and spend control.

When using **Claude**, default to the current line below (choose the tier by task); equivalent
tiers exist across providers.

| Model (Claude) | Model ID | Use it for |
|---|---|---|
| Claude Opus 4.8 | `claude-opus-4-8` | Most capable Opus-tier; complex reasoning, agents, coding, hard tasks. |
| Claude Sonnet 5 | `claude-sonnet-5` | Best speed/intelligence balance; near-Opus quality on coding/agentic at lower cost — high-volume production paths. |
| Claude Haiku 4.5 | `claude-haiku-4-5` | Fast, cheap; simple classification/extraction/short responses. |
| Claude Fable 5 | `claude-fable-5` | Only when explicitly chosen — most demanding long-horizon reasoning; premium pricing. |

Never silently downgrade for cost — that's a documented decision. Model IDs, providers, and
capabilities change; verify against the provider's current docs rather than memory before
shipping. **Compliance across providers:** any provider that touches personal data, PHI, or
regulated data needs a DPA, acceptable data-residency, and no-training terms (Principles IV &
VIII) — do not route sensitive data to a provider or gateway not contracted for it.

### 2. Right-size the surface (simplest tier that works)

- **Single call** — classification, extraction, summarization, Q&A, generation.
- **Workflow** — multi-step pipelines with code-controlled logic and tool use you orchestrate.
- **Agent** — only for genuinely open-ended, model-driven tool use where the outcome justifies
  the cost and errors are recoverable. Prefer the SDK's tool runner over a hand-rolled loop; reach
  for a hosted/managed agent only when you need server-run loops or per-session sandboxes.

Don't build an agent where a single call or a deterministic workflow does the job.

### 3. Build standards (defaults, unless the plan overrides)

- **Provider-neutral integration.** Call models through the Vercel AI SDK (`ai`), the provider's
  official SDK, or AI Gateway — never an untrusted OpenAI-compatible shim. Keep the model/provider
  swappable rather than hard-wired through the app.
- **Use the model's reasoning controls** where available (extended/adaptive thinking, effort or
  verbosity settings) for non-trivial tasks, rather than fixed hacks.
- **Stream** any response with long input/output (e.g. `streamText`) to avoid timeouts.
- **Structured outputs** (schema-constrained, e.g. `generateObject` or the provider's structured
  outputs) whenever the app parses the result — never regex a free-text blob. Validate tool
  inputs; parse tool JSON, don't string-match it.
- **Cache** large stable prefixes (system prompt, retrieved context) where the provider supports
  prompt caching, to cut cost and latency; keep the cached prefix byte-stable (no timestamps/UUIDs
  up front).
- **Keys server-side only.** LLM calls run in server actions / route handlers (or via AI Gateway);
  provider keys never reach the client. Never put secrets, PHI, or PII in prompts, system messages,
  or logs — they persist in traces and history.
- **Ground and cite** for RAG/agents: retrieve, pass context explicitly, prefer answers grounded
  in sources over model memory; show provenance where users act on it.
- **Handle refusals and failures** as first-class outcomes (check `stop_reason`); degrade
  gracefully, never surface a raw stack trace or an empty bubble.

### 4. Conversational agents & assistants (capability contract)

A conversational/agentic capability is not "done" until its plan states, and its implementation
enforces: the **model + effort**; the **persona and scope** (what it does, and explicit
out-of-scope topics it must decline); the **tools/functions** it can call and their permissions
(read vs. mutating, and which mutations need confirmation); the **grounding/knowledge sources**;
the **safety guardrails and escalation-to-human** path; and **evaluation** against representative
transcripts. Scope must fit the **industry**: regulated domains (health, finance, legal) require
tighter scope, disclaimers, human escalation, and the Principle VIII triage — an assistant must
not give regulated advice it isn't authorized to give.

### 5. Evaluation & observability

Every AI capability ships with a lightweight **eval set** (representative inputs + expected
qualities or a rubric) run before launch and after prompt/model changes — prompt edits are
regressions-in-waiting. Log prompts/outputs for debugging **with PII redacted**; track token
usage and latency. Treat prompts and model IDs as **versioned artifacts**: a model or prompt
change is a reviewable change, not a silent tweak. Evals use a versioned dataset, pre-agreed
rubrics and pass thresholds, and are compared against an approved baseline; deterministic
behavior around the model (tool authorization, output validation, fallbacks) is covered by
ordinary software tests with fixed model responses (Verification Standards §7,
`.specify/standards/ai-evaluation.md`).

## Verification Standards

Companion clauses to Principle V. They state the durable outcomes; operational recipes,
configuration, and examples live in `.specify/standards/` (see Supporting Standards). Each clause
applies where the project type and triage make it relevant — the plan states which apply and
justifies any exclusion.

### 1. Acceptance criteria traceability

Every spec lists its behavioral acceptance criteria with stable IDs and verification methods
(Principle V). Use one row per criterion and allow several layers. At completion, record the
test path/title and CI run, or the manual/eval report. Manual evidence MUST name the reviewer,
environment, date, steps, and result.

| ID | Method(s) | Criterion | Evidence |
|---|---|---|---|
| AC-01 | `[E2E]` | Valid password-reset link permits a new password. | `tests/e2e/auth/password-reset.spec.ts` |
| AC-02 | `[INTEGRATION]` | Expired reset token is rejected. | `tests/integration/auth/reset-token.test.ts` |
| AC-03 | `[UNIT]` | Passwords below the policy threshold are rejected. | `password-policy.test.ts` |
| AC-04 | `[INTEGRATION]` `[E2E]` | Tenant A cannot read or edit tenant B data. | Direct API denial + protected UI journey |
| AC-05 | `[MANUAL]` | Complex dialog is usable with a screen reader. | Named reviewer + steps + supported AT/browser |
| AC-06 | `[EVAL]` | Answers meet approved groundedness thresholds. | Versioned dataset + scored evaluation report |

The plan records expected outcomes and thresholds **before** implementation. Tasks pair delivery
with verification — e.g. "Implement AC-04 server authorization and add cross-tenant read/write
denial tests." Component checks use `[UNIT]` or `[INTEGRATION]` according to their boundary;
visual checks are recorded under `[E2E]` or `[MANUAL]`.

### 2. Security behavior must be tested

Protected capabilities MUST prove rejection of unauthenticated requests, cross-user and
cross-tenant reads and writes, and lower-privilege actions. Tests MUST bypass UI controls and
call the server boundary directly. Verify malformed inputs, expired/revoked credentials,
sensitive-field exposure, duplicate submissions, and idempotency. Add replay, webhook-signature,
rate-limit, and injection cases when applicable. Assert both the rejection **and** the absence of
forbidden side effects. Hiding the button is never an authorization test. (Extends Principle IV.)

### 3. Complete state coverage

Specifications MUST identify all materially reachable states and transitions: initial, loading,
loaded, empty, validation failure, recoverable error, fatal error, permission denied,
unauthenticated, stale/expired, partial success, and offline/network failure where relevant.
Plans MUST assign verification to the applicable states and justify exclusions. Include
retry/recovery, cancellation, repeated actions, and concurrent changes where they affect
correctness. Assertions MUST verify retained data, feedback, and the next permitted actions.
(Extends Design Rules §3.)

### 4. API and contract testing

Important boundaries MUST have executable verification where practical: request/response
schemas, status and error semantics, webhook payloads, events/messages, import/export formats,
provider adapters, and database invariants. Include invalid payloads, permissions, pagination,
and compatibility with supported clients. Mocks MUST conform to the actual versioned contract.
Validate provider assumptions through sandbox or contract checks; include webhook signature and
replay tests where relevant.

### 5. Database change safety

Migrations MUST work against existing data and a production-like schema, not only an empty
database. Before deployment, test: upgrade from the supported prior version with representative
synthetic or sanitized data; backfills, new constraints, row counts, and critical invariants;
and compatibility during rolling deployment. Check locking and runtime for high-risk changes at
representative scale.

Destructive changes MUST have an explicit recovery strategy. Prefer **expand → migrate →
contract** where old and new application versions coexist. Verify resumability/idempotency of
backfills when required. Document whether recovery uses rollback, roll-forward, or restore — do
not assume every migration is reversible. High-risk changes MUST exercise the relevant recovery
procedure and confirm no silent data loss. (Extends Principle IV.)

### 6. Observability is part of correctness

Critical operations SHOULD emit structured telemetry identifying the operation, time,
correlation/request ID, outcome, and failure stage. Logs MUST exclude secrets and unnecessary
personal data. Tests SHOULD trigger representative error paths, verify expected telemetry and
redaction, and validate alert delivery in a controlled environment. Prevent test traffic from
paging production responders. Verify important analytics/conversion events fire once per
intended action where relevant.

### 7. AI software tests versus AI evaluations

AI features need both, and one never substitutes for the other.

- **Software tests** MUST verify deterministic behavior: request construction, tool
  authorization, structured-output validation, persistence, retries, fallbacks, guardrail
  routing, timeouts, and error handling. Use fixed model responses for ordinary CI. Include
  invalid output, provider failure, and denied tool execution. Authorization MUST be enforced
  outside the model.
- **Evaluations** MUST measure model-dependent behavior against a versioned, representative
  dataset: correctness, groundedness, relevance, hallucination, refusal behavior, safety, and
  domain quality. Include adversarial and risky cases such as prompt injection and unauthorized
  tool requests. Define scoring rubrics, pass thresholds, failure severity, and human
  adjudication for ambiguous results **before** running the evaluation.

Record the model/version, prompt version, retrieval configuration, tool policy, dataset version,
and results. Relevant evals MUST run before releasing changes to these inputs. Compare with the
approved baseline, investigate regressions, and repeat samples where variance matters. Track
latency and cost where product requirements depend on them. A plausible response from a manual
demonstration is not an evaluation.

### 8. Evidence record

Completion evidence for each criterion is recorded in a consistent shape, for example:

```text
AC:          AC-04
Methods:     [INTEGRATION, E2E]
States:      unauthenticated, forbidden, permitted
Environment: isolated preview for candidate commit
Evidence:    test IDs + CI run + report artifact
Result:      PASS
Exception:   none
```

### Supporting Standards

Implementation MUST follow the versioned standards in `.specify/standards/`. Plans MUST identify
the applicable standards and document approved, time-bounded exceptions.

| File | Covers |
|---|---|
| `testing.md` | Hierarchy, traceability, tagging, pipeline sequence, Definition of Done checklist |
| `playwright.md` | Locators, assertions, fixtures, auth state, folder layout, CI config |
| `security-testing.md` | Negative-path matrix for auth, tenancy, input, and webhooks |
| `data-migrations.md` | Upgrade, backfill, and recovery checks |
| `ai-evaluation.md` | Datasets, rubrics, thresholds, and eval records |

This constitution defines required outcomes, enforcement, and exception governance; the standards
hold evolving recipes, configuration, folder conventions, browser matrices, and artifact
retention. Keep one authoritative copy of each operational rule and link to it rather than
duplicating it. Standards changes are reviewed with the same care as the code they affect.

**Exceptions.** An alternative framework, a quarantined test, or any other departure from a
MUST is recorded in the plan with its rationale, owner, approver, compensating evidence, and
expiry. Expired exceptions fail review.

## Design Rules

Design standards for the v0 + Tailwind + shadcn/Radix stack so every surface — marketing site,
app, or content site — reads as one intentional product, not assembled fragments. These apply to
**whoever implements the UI — v0 or Claude Code** — so the output stays coherent and either tool
can extend it safely.

### 1. Design system first

- **Tokens, not magic values.** Colors, spacing, radii, shadows, and typography come from
  `tailwind.config.ts` design tokens and Tailwind scale utilities — no ad-hoc hex codes or
  one-off pixel values when a token/utility exists. New tokens are added deliberately, not inlined.
- **Reuse before rebuild.** Compose existing shadcn/Radix components and variants
  (`class-variance-authority`) before creating new ones; one component per concept, not five
  near-duplicates.

### 2. Visual language

- **Typographic scale.** A defined, limited type scale (few sizes/weights); clear hierarchy;
  comfortable line-height and measure (~60–75 chars for body). Avoid generic defaults — no Inter/
  Roboto/Arial-by-accident; choose type that fits the brand and use it consistently.
- **Spacing & layout.** Consistent spacing scale and a grid; generous, rhythmic whitespace;
  alignment and proximity express grouping. No cramped or arbitrary gaps.
- **Color & theme.** A cohesive, limited palette with defined semantic roles (primary, accent,
  muted, destructive, success) and accessible foreground/background pairings; light **and** dark
  themes honored via tokens/`next-themes`. Avoid cliché AI aesthetics (e.g. purple-on-dark
  gradients) unless the brand calls for it.
- **Depth & motion.** Elevation and motion are purposeful and restrained; animations (framer-
  motion) are quick, easing-based, and respect `prefers-reduced-motion`. Motion clarifies state
  and hierarchy — it is never decoration for its own sake.

### 3. Every state is designed

Design and implement the full set of states, not just the happy path: **default, hover, focus,
active, disabled, loading/skeleton, empty, and error**. Interactive elements have visible focus
rings and adequate hit targets. Forms show inline validation and preserve input on error.

### 4. Responsive & consistent

- **Mobile-first, fluid.** Layouts work from small screens up using relative units and Tailwind
  breakpoints; no fixed-width desktop-only designs; images via `next/image` with correct sizing.
- **Consistency is a feature.** The same action looks and behaves the same everywhere; button
  hierarchy (primary/secondary/ghost), iconography, and copy tone are uniform across the product.

### 5. Accessible by construction

A11y from Principle VII is a design rule too: semantic HTML, labeled controls, keyboard operability,
logical focus order, and WCAG AA contrast are acceptance criteria. Prefer Radix primitives for
interactive controls to get accessibility for free rather than reimplementing it.

## Development Workflow

1. `/speckit-constitution` — confirm/adjust these principles and declare the **project type**.
2. `/speckit-specify` — capture the feature's what/why, acceptance criteria with stable IDs and
   verification methods, and the applicable states (and data/auth shape for applications), and
   run the **compliance & privacy triage** (record the result).
3. `/speckit-clarify` (optional) — de-risk ambiguity before planning.
4. `/speckit-plan` — architecture and stack decisions consistent with this constitution;
   revisit the triage, record the data classification, the risk assessment, the **verification
   plan** (test layer, states, environment, and evidence per criterion; browsers/viewports;
   applicable standards and exceptions), and — for AI features — the pinned model/effort and
   capability contract (AI Engineering Standards).
5. `/speckit-tasks` — ordered, independently shippable tasks that pair each implementation task
   with its verification work (including risk/compliance mitigations and AI eval tasks as
   explicit tasks).
6. `/speckit-analyze` / `/speckit-checklist` (optional; **`/speckit-checklist` required for
   escalated projects**) — verify cross-artifact consistency and compliance-control coverage.
7. **Choose the implementer** (Principle II) — for each feature/task, decide who builds it:
   - **v0** — fast in-browser iteration, visual/preview-driven work, and full-stack scaffolds you
     want to see running quickly (marketing surfaces, CRUD screens, prototypes, straightforward
     server actions/routes).
   - **Claude Code** — complex or cross-cutting logic, large refactors, precise control, test
     authoring, tricky integrations/migrations, and anything driven from the repo/CLI.
   - **Split** — e.g. v0 builds the surface and scaffolds the endpoints, Claude Code hardens the
     logic, security, and tests. Either tool can also own an entire feature end to end.
   Record the choice in the plan; it can differ per feature and change over time.
8. `/speckit-implement` (Claude Code) and/or build in **v0** — implement against the tasks;
   keep build/lint/typecheck/tests green whoever builds.

**Delivery flow:** Spec → Clarify → Plan → Tasks → Implement → Test → Review → Ship. Spec
defines behavior, criteria, and applicable states. Clarify resolves ambiguity and risk. Plan
assigns test layers, fixtures, environments, and release gates. Tasks include verification
alongside implementation. Implement builds behavior and tests together. Test runs the required
suites and records evidence. Review checks evidence against every criterion and this
constitution. Ship deploys the approved candidate, runs safe smoke checks, and verifies
operational signals. Failed evidence returns work to the relevant earlier step.

Quality gates before merge (the Definition of Done in Principle V): CI green (build, lint,
typecheck, unit/integration tests, Playwright critical journeys, no unresolved flaky results),
every acceptance criterion mapped to evidence, **code-reviewed**, input validated and mutations
authorized (applications), no secrets committed, a11y and responsive checks pass, design rules
upheld, SEO/metadata present (marketing/content), AI capabilities evaluated against their eval set
with keys server-side (AI features), and — for escalated projects — the compliance checklist
satisfied and privacy controls in place. UI changes are visually reviewed on the Vercel preview
before merge; production monitoring is wired for applications. Escalated projects also require the
§6 sign-off before production launch.

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
stay consistent with this document. The standards in `.specify/standards/` are subordinate to
this constitution: they may add operational detail but never weaken a MUST here; a conflict is
resolved by fixing the standard or amending this document. Exceptions to a MUST follow the
Exceptions rule under Supporting Standards (rationale, owner, approver, compensating evidence,
expiry).

**Version**: 2.0.0 | **Ratified**: 2026-07-08 | **Last Amended**: 2026-10-13
