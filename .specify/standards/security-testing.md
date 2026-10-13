# Security Testing Standard

**Version**: 1.0.0 | **Last Updated**: 2026-10-13 | **Implements**: Constitution Principle IV,
Verification Standards §2 and §4

Security behavior is proven at the server boundary. Tests call the route handler, server action,
or API directly — bypassing the UI — and assert both the rejection **and** the absence of any
forbidden side effect (no row written, no email sent, no event emitted). Hiding a button is never
an authorization test.

## 1. Negative-path matrix

For every protected capability, fill in one row per applicable case. Mark non-applicable cases
`N/A` with a reason.

| Case | Expected result | Side-effect check | Layer |
|---|---|---|---|
| Unauthenticated request | 401 / redirect to sign-in | No mutation | `[INTEGRATION]` |
| Authenticated, wrong user (same tenant) | 403 / 404 | No mutation, no data leak | `[INTEGRATION]` |
| Cross-tenant read | 403 / 404 | Response contains no foreign data | `[INTEGRATION]` |
| Cross-tenant write | 403 / 404 | Foreign record unchanged | `[INTEGRATION]` |
| Lower-privilege role performs privileged action | 403 | No mutation | `[INTEGRATION]` |
| Malformed / oversized / wrong-type input | 400 / 422 with safe message | No mutation | `[INTEGRATION]` |
| Expired credential or session | 401 | No mutation | `[INTEGRATION]` |
| Revoked credential or session | 401 | No mutation | `[INTEGRATION]` |
| Sensitive fields in response (hashes, tokens, internal IDs, PII) | Not present | — | `[INTEGRATION]` |
| Duplicate submission | Idempotent result | Exactly one effect | `[INTEGRATION]` |
| Replayed request / webhook | Rejected or idempotent | Exactly one effect | `[INTEGRATION]` |
| Invalid webhook signature | 400 / 401 | No processing | `[INTEGRATION]` |
| Rate limit exceeded | 429 | No further effects | `[INTEGRATION]` |
| Injection (SQL, prompt, HTML/script) | Treated as data | No escalation, escaped output | `[INTEGRATION]` / `[E2E]` |

Add at least one `[E2E]` journey per critical protected flow that confirms the UI handles a
denial correctly (message shown, no broken state). The UI test supplements, never replaces, the
direct boundary test.

## 2. Fixtures

- Create at least two tenants and, within one tenant, at least two users of different roles.
- Use unique run identifiers so parallel workers never share mutable records.
- Generate expired and revoked credentials deterministically (controlled clock, explicit
  revocation), not by waiting.

## 3. Contracts

- Validate request and response bodies against the versioned schema (e.g. the same Zod schema
  used at the boundary, or OpenAPI).
- Cover status and error semantics, pagination limits, and compatibility with supported clients.
- Stubs and mocks MUST conform to the current versioned provider contract; validate the
  assumptions through the provider's sandbox or a contract test.
- Payments use the provider's test mode only.

## 4. Row-level security

Where the database enforces row-level security, test the policies directly with each role
(including `anon`), in addition to the application-layer checks.
