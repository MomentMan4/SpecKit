# Playwright Standard

**Version**: 1.0.0 | **Last Updated**: 2026-10-13 | **Implements**: Constitution Principle V
(Default framework, Resilient browser tests, Authentication and environment, Failure evidence,
Browser and viewport coverage, Accessibility and selective visual regression)

Playwright is the default browser E2E framework. An alternative requires a recorded plan
exception. Pin a compatible `@playwright/test` version in the lockfile and use matching browser
binaries in CI.

References: [Best Practices](https://playwright.dev/docs/best-practices) ·
[Authentication](https://playwright.dev/docs/auth) ·
[Configuration](https://playwright.dev/docs/test-configuration)

## 1. Locator priority

1. `getByRole` with accessible name
2. `getByLabel` for form controls
3. Meaningful `getByText`, `getByAltText`, or `getByPlaceholder` where appropriate
4. Stable `getByTestId` where semantics are insufficient
5. CSS/XPath fallback — only with a documented, unavoidable reason

Scope ambiguous locators to a meaningful region (e.g. `page.getByRole('dialog')`). Do not use
`first()` or `nth()` merely to hide ambiguity. Placeholder text does not replace an accessible
label.

## 2. Assertions and waiting

- Assert user-visible outcomes with web-first assertions (`await expect(locator)…`), which retry.
- Rely on locator auto-waiting. Wait for meaningful UI state, or register a response/event wait
  **before** the action that triggers it.
- Never commit `page.waitForTimeout()` as synchronization, and do not use a broad
  `networkidle` wait as a readiness signal. Remove diagnostic sleeps before merge.

```ts
import { test, expect } from '@playwright/test';

test('AC-07 reset form validates input @critical @auth', async ({ page }) => {
  await page.goto('/forgot-password');

  await page.getByLabel('Email', { exact: true }).fill('invalid-address');
  await page.getByRole('button', { name: 'Send reset link' }).click();

  await expect(page.getByRole('alert')).toContainText('Enter a valid email');
});
```

Adapt routes, accessible names, and feedback to the real specification. This example proves form
validation only; a full reset criterion also needs the successful journey and server-side token
verification.

## 3. Folder layout

```text
tests/
  unit/                      # or colocated *.test.ts
  integration/
  e2e/
    auth/
    onboarding/
    billing/subscription-cancellation.spec.ts
  fixtures/                  # deterministic data and roles
  helpers/                   # small reusable actions
  setup/                     # environment and auth setup
evals/                       # datasets, rubrics, runners
playwright/.auth/            # gitignored sensitive state
playwright.config.ts
```

- Organize E2E tests by capability and journey, not by page file.
- Name each test for its criterion and expected behavior (`'AC-04 tenant A cannot edit tenant B
  invoice @critical @auth'`).
- Keep helpers small; assertions stay in the test where they are easy to inspect.

## 4. Fixtures and data isolation

- Fixtures own setup and cleanup and fail clearly when prerequisites cannot be established.
- Create known users, roles, and records per test or per worker; use a unique run identifier in
  every created record.
- Use a dedicated account per parallel worker (or per test) wherever shared state can change.
  Browser-context isolation does not isolate the database.
- Control clocks (`page.clock`), time zones (`timezoneId`), randomness, and seed data where they
  affect results.
- Clean up safely or use disposable environments. Seed/reset endpoints MUST be unreachable in
  production.
- Never use production customer data.

## 5. Authentication

- Use a setup project to sign in and save `storageState` per role, and declare it as a
  dependency of authenticated projects. Keep unauthenticated coverage in its own project.
- Store state only in `playwright/.auth/` and add it to `.gitignore`. Use dedicated
  non-production accounts. Never upload saved state as a public artifact.
- Regenerate state per run as needed.
- Login, logout, session expiry, and revocation are tested explicitly — saved state is not a
  substitute for testing them.

```gitignore
playwright/.auth/
test-results/
playwright-report/
```

## 6. Baseline configuration

Starter config for an already-running preview environment. It deliberately requires a base URL.

```ts
import { defineConfig, devices } from '@playwright/test';

const baseURL = process.env.E2E_BASE_URL;
if (!baseURL) throw new Error('E2E_BASE_URL is required');

export default defineConfig({
  testDir: './tests/e2e',
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { open: 'never' }],
    ['junit', { outputFile: 'test-results/junit.xml' }],
  ],
  use: {
    baseURL,
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure',
    video: 'off',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
});
```

Notes:
- Retries alone do **not** reject a retry-pass. Enable the pinned version's fail-on-flaky option
  (e.g. `failOnFlakyTests: !!process.env.CI` where supported) or add a CI step that fails when the
  report contains flaky results.
- Start with one CI worker. Increase concurrency or shard only after data isolation is proven.
- Add setup-project dependencies and role-specific `storageState` only for authenticated
  projects.
- Add `Mobile Safari`/`Pixel` device projects when the product is mobile-critical, and Firefox/
  WebKit projects when support commitments or risk require them (PR or main/nightly per plan).

## 7. Running suites

```bash
npx playwright test --project=chromium --grep @critical   # PR gate
npx playwright test --grep @smoke                         # deployment usability
npx playwright test --grep @regression                    # main / nightly
```

Use `pnpm exec playwright …` in pnpm projects.

## 8. Diagnostics and artifacts

- Always publish the HTML and JUnit reports, even when tests fail.
- Retain traces and screenshots on failure; capture console and page errors and fail on
  unexpected ones for critical journeys.
- Video only where it adds diagnostic value.
- Restrict artifact access and set retention (e.g. 14 days, subject to data policy). Traces can
  contain tokens, personal data, and request bodies.

## 9. Accessibility and visual checks

- Critical journeys assert keyboard operation and focus behavior (Tab order, focus moves into
  and out of dialogs, Escape closes overlays).
- Major pages SHOULD run an automated scan (e.g. `@axe-core/playwright`). A clean scan is not
  complete accessibility evidence — pair it with `[MANUAL]` review for names, semantics, contrast,
  and complex widgets.
- Visual snapshots (`toHaveScreenshot`) only for stable, high-value surfaces. Fix fonts, disable
  animations, mask timestamps and dynamic data, and pin OS/browser for baselines. Baseline
  updates are reviewed against intended design changes, never accepted just to clear a failure.
