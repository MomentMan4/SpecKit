# Testing Standard

**Version**: 1.0.0 | **Last Updated**: 2026-10-13 | **Implements**: Constitution Principle V,
Verification Standards §1–§8

Operational companion to the constitution. The constitution states the required outcomes; this
file holds the recipes. Where they differ, the constitution wins — fix this file.

Key words: **MUST** is a requirement; **SHOULD** permits a justified, recorded exception; **MAY**
is optional.

## 1. Choosing the layer

Use the lowest-cost layer that reliably proves the behavior.

| Behavior | Layer | Method tag |
|---|---|---|
| Business rules, calculations, transformations, validators, policy decisions | Unit | `[UNIT]` |
| Data access, DB invariants, APIs, server actions, authorization, webhooks, provider adapters | Integration | `[INTEGRATION]` |
| Complex component interaction | Component (unit or integration by boundary) | `[UNIT]` / `[INTEGRATION]` |
| User journeys across browser, navigation, auth, persistence | Playwright E2E | `[E2E]` |
| Visual regression | Playwright snapshots or human review | `[E2E]` / `[MANUAL]` |
| Screen-reader usability, complex a11y, real-device checks | Human review | `[MANUAL]` |
| Model-dependent AI quality | Evaluation run | `[EVAL]` |

Do not repeat exhaustive lower-level cases in E2E. Do not mock away the boundary a test claims
to prove.

## 2. Critical journeys

A journey is critical when its failure:
- blocks product access or primary value;
- corrupts important data;
- violates privacy or authorization;
- affects billing or account state; or
- causes an irreversible or materially harmful action.

Typical candidates: login, onboarding, primary CRUD, protected actions, checkout, uploads, and
critical AI workflows. Every critical journey MUST have Playwright E2E coverage, MUST be tagged
`@critical`, and server-side and tenant boundaries MUST also be proven by integration tests.

## 3. Traceability

Spec — every behavioral criterion has an ID and method(s):

```text
AC-01 [E2E]               Valid password-reset link permits a new password.
AC-02 [INTEGRATION]       Expired reset token is rejected.
AC-04 [INTEGRATION][E2E]  Tenant A cannot read or edit tenant B data.
```

Plan — one row per criterion:

| AC | Layer(s) | States | Environment | Evidence required |
|---|---|---|---|---|
| AC-04 | Integration + E2E | unauthenticated, forbidden, permitted | Isolated preview | Test IDs + CI run + report |

Tasks — pair delivery with verification: *"Implement AC-04 server authorization and add
cross-tenant read/write denial tests."*

Completion — evidence record per criterion:

```text
AC:          AC-04
Methods:     [INTEGRATION, E2E]
States:      unauthenticated, forbidden, permitted
Environment: isolated preview for candidate commit
Evidence:    test IDs + CI run + report artifact
Result:      PASS
Exception:   none
```

Manual evidence MUST name the reviewer, environment, date, steps, and result. Eval evidence MUST
name the dataset version and the scored report. Unmapped criteria are incomplete.

## 4. State checklist

For each feature, mark which states apply and how each is verified; justify exclusions.

- [ ] Initial
- [ ] Loading
- [ ] Loaded
- [ ] Empty
- [ ] Validation failure
- [ ] Recoverable error (with retry/recovery)
- [ ] Fatal error
- [ ] Permission denied
- [ ] Unauthenticated
- [ ] Stale / expired
- [ ] Partial success
- [ ] Offline / network failure
- [ ] Cancellation, repeated action, concurrent change (where correctness depends on them)

Assertions verify retained data, user feedback, and the next permitted actions.

## 5. Tags and selection

| Tag | Meaning |
|---|---|
| `@smoke` | Fastest checks that a deployment is usable |
| `@critical` | Business-critical journeys (required on every PR) |
| `@regression` | Broader coverage, including defect cases |
| `@auth`, `@billing`, … | Domain tags for risk-based selection |

Tags may overlap. CI MUST verify that each selected suite actually contains and executes the
required tests (an empty grep is a failure, not a pass). Production smoke uses a separate,
explicitly safe selection — never the whole `@smoke` suite by assumption.

## 6. Pipeline sequence

- **PR**: install locked dependencies and matching browsers → lint, typecheck, unit, and
  applicable integration tests → build → deploy/start the candidate → wait for a meaningful health
  check → seed isolated data → run Chromium `@critical` plus relevant regression tests. Add mobile
  coverage when required.
- **Main or nightly**: broader regression suite and the supported browser/viewport matrix;
  sandbox/contract checks and scheduled evals according to risk. Failures create owned
  remediation work.
- **Release**: require outstanding matrix, security, migration, and relevant eval evidence for
  the candidate. Run safe deployment smoke checks and apply the documented rollback or
  stop-release criteria.
- **Always**: publish reports and failure artifacts even after failures; clean up isolated
  resources; enforce artifact access and retention rules (e.g. 14 days, subject to data policy).
  Missing artifacts or skipped required tests must be visible.

A scheduled job never substitutes for a required pre-release check.

## 7. Regression and flake handling

- Confirmed defect → regression test at the lowest appropriate layer, before or alongside the
  fix. Confirm it fails without the fix when feasible.
- Flaky test → open an issue and investigate as a defect. A retry-pass is reported as flaky and
  does not satisfy the gate.
- Quarantine → requires owner, issue link, reason, expiry date, and compensating coverage.
  Critical journeys never silently lose verification.
- Longer waits, more retries, or bigger timeouts without a root cause are not fixes.

## 8. Right-sized expectations

| Project type | Minimum |
|---|---|
| Application | Meaningful unit tests; data, auth, and API integration tests; Playwright critical journeys; negative paths; regression coverage |
| Marketing website | Navigation and conversion smoke tests; form validation/submission; responsive behavior; accessibility; key analytics events where practical |
| Content website | Rendering/navigation smoke checks; applicable link/form checks; responsive behavior; accessibility |

## 9. Definition of Done checklist

- [ ] **Specification and evidence** — every acceptance criterion mapped, implemented, and
  verified; manual/eval evidence attached where applicable.
- [ ] **Automated quality** — lint, typecheck, build, required unit/integration/E2E, and
  release-matrix checks pass on the candidate; no unresolved flaky result or silently skipped
  required test.
- [ ] **Risk and state coverage** — authorization denials, tenant isolation, applicable UI
  states, contracts, migration/backfill safety, and recovery verified.
- [ ] **User experience** — critical journeys exercised in a deployable environment; responsive,
  accessibility, and selected visual checks complete; no unexplained console/browser/server
  errors.
- [ ] **Regression and AI** — confirmed bugs have regression evidence; relevant AI evals meet
  pre-agreed thresholds without unacceptable regressions.
- [ ] **Operational readiness** — telemetry and redaction checked; documentation and
  `.env.example` updated; diagnostics retained securely; deployment/recovery plan reviewed.
- [ ] **Compliance** (escalated projects) — compliance checklist satisfied and privacy controls in
  place.
- [ ] **Review** — reviewer confirms evidence and scope; every exception names its approver,
  rationale, owner, compensating control, and expiry.
