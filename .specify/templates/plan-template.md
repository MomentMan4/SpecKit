# Implementation Plan: [FEATURE]

**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]

**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: [e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION]

**Primary Dependencies**: [e.g., FastAPI, UIKit, LLVM or NEEDS CLARIFICATION]

**Storage**: [if applicable, e.g., PostgreSQL, CoreData, files or N/A]

**Testing**: [unit/integration runner, e.g., Vitest; browser E2E: Playwright (or a justified alternative recorded under Exceptions) or NEEDS CLARIFICATION]

**Target Platform**: [e.g., Linux server, iOS 15+, WASM or NEEDS CLARIFICATION]

**Project Type**: [e.g., library/cli/web-service/mobile-app/compiler/desktop-app or NEEDS CLARIFICATION]

**Performance Goals**: [domain-specific, e.g., 1000 req/s, 10k lines/sec, 60 fps or NEEDS CLARIFICATION]

**Constraints**: [domain-specific, e.g., <200ms p95, <100MB memory, offline-capable or NEEDS CLARIFICATION]

**Scale/Scope**: [domain-specific, e.g., 10k users, 1M LOC, 50 screens or NEEDS CLARIFICATION]

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

[Gates determined based on constitution file]

Verification gates (Principle V and Verification Standards) — all must be answered:

- [ ] Every behavioral acceptance criterion in the spec has an ID and verification method(s)
- [ ] Critical journeys identified and assigned Playwright E2E coverage (`@critical`)
- [ ] Server-side authorization and tenant boundaries covered by integration tests (not UI only)
- [ ] Applicable states assigned verification; exclusions justified
- [ ] External integrations have a stub + contract/sandbox strategy
- [ ] Migrations (if any) classified with upgrade and recovery checks
- [ ] AI features (if any) have deterministic software tests **and** an eval plan with thresholds
- [ ] Supported browsers/viewports named
- [ ] Applicable `.specify/standards/` files listed; exceptions recorded below

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
# [REMOVE IF UNUSED] Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [REMOVE IF UNUSED] Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [REMOVE IF UNUSED] Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure: feature modules, UI flows, platform tests]
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

## Verification Plan

<!--
  Constitution: Principle V, Verification Standards §1. One row per acceptance criterion;
  record expected outcomes and thresholds BEFORE implementation.
-->

**Applicable standards**: [e.g., testing.md, playwright.md, security-testing.md, data-migrations.md, ai-evaluation.md]

**Browsers / viewports**: [e.g., Chromium desktop on PR; + Mobile Safari, Firefox on nightly]

**Test environment**: [e.g., isolated Vercel preview for the candidate commit, seeded per run]

| AC | Layer(s) | States covered | Environment | Evidence required |
|----|----------|----------------|-------------|-------------------|
| AC-01 | E2E | loaded, validation failure | Preview | `tests/e2e/...` + CI run |
| AC-04 | Integration + E2E | unauthenticated, forbidden, permitted | Preview | Test IDs + CI run + report |

**CI gates**: [PR: lint, typecheck, unit, integration, build, Playwright @critical; main/nightly: @regression + matrix; release: ...]

### Exceptions

> Alternative frameworks, quarantined tests, or any other departure from a constitution MUST.

| Exception | Rationale | Owner | Approver | Compensating evidence | Expiry |
|-----------|-----------|-------|----------|-----------------------|--------|

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
