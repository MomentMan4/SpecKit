# Data Migration Standard

**Version**: 1.0.0 | **Last Updated**: 2026-10-13 | **Implements**: Constitution Principle IV,
Verification Standards §5

Migrations must work on existing data, not only on an empty database.

## 1. Classify the change

| Risk | Examples | Required checks |
|---|---|---|
| Low | Add nullable column, add table, add index concurrently on small table | Upgrade test on seeded data |
| Medium | Backfill, new `NOT NULL`/unique/foreign-key constraint, column type change | + invariant and row-count checks, rolling-deploy compatibility |
| High | Drop/rename column or table, large backfill, lock-heavy DDL on large tables, data rewrite | + recovery procedure exercised, runtime/locking measured at representative scale |

Record the classification in the plan.

## 2. Pre-deployment checks

- [ ] Upgrade from the supported prior version succeeds on representative synthetic or sanitized
  data (never production personal data).
- [ ] Backfills produce the expected values; row counts before and after are reconciled.
- [ ] New constraints hold on existing data (find violators before applying).
- [ ] Critical invariants are asserted after migration (e.g. every invoice has an owner).
- [ ] Old and new application versions both work against the migrated schema during a rolling
  deploy.
- [ ] High-risk changes: locking and runtime measured at representative scale.
- [ ] Backfills are resumable and idempotent when they may be interrupted.

## 3. Expand → migrate → contract

When old and new application versions coexist:

1. **Expand** — add the new column/table; write to both old and new.
2. **Migrate** — backfill; switch reads to the new shape; verify.
3. **Contract** — remove the old shape in a later release, once nothing reads it.

## 4. Recovery

Every destructive change documents its recovery strategy:

- **Rollback** — a tested down-migration (only when it truly restores data).
- **Roll-forward** — a corrective migration, prepared in advance.
- **Restore** — point-in-time recovery or backup; restore time known.

Do not assume every migration is reversible. High-risk changes exercise the chosen recovery path
in a non-production environment and confirm no silent data loss.

## 5. Evidence

Attach to the plan or PR: the classification, the checks run (with environment and dataset), row
counts/invariant results, and the recovery strategy with its test result.
