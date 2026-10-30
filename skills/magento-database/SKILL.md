---
name: magento-database
description: Diagnose Magento database queries, locks, schema changes, and data repairs. Use when the task centers on MySQL/MariaDB behavior or a database intervention rather than routine PHP persistence code.
---

# Magento database work

## Gather evidence

Identify engine/version, table prefix, relevant table definitions, indexes, row counts, and the failing query or lock evidence. Inspect store-scoped EAV values and the actual entity link field; do not assume `entity_id` and `row_id` are interchangeable across editions and schemas.

Use existing diagnostics or authorized read-only queries. Begin with query plans and bounded inspection. `EXPLAIN ANALYZE` executes the statement; assess its cost and supported syntax before using it on a live database.

## Design the intervention

- Choose indexes from actual predicates, joins, ordering, and selectivity. Account for write cost, existing indexes, and measured plans instead of adding indexes speculatively.
- For persistent module schema changes, use `db_schema.xml` where supported; use data patches for application migrations. Check installed schema, dependencies, and release-specific whitelist requirements.
- Prefer application services when a data change requires validation, events, indexers, or cache invalidation. If direct SQL is appropriate, state which application effects must be reconciled.
- For `UPDATE` or `DELETE`, provide a matching `SELECT` preview and count using the same predicates. Bound large writes with stable-key batches and explain locking and transaction boundaries. Parameterize dynamic values.
- Assess DDL locking and engine capabilities separately from DML. Do not promise transactional rollback for auto-committing DDL or recovery of deleted data without a retained copy or backup. Include post-change validation and a realistic recovery path for mutations.

## Execution and delivery

Prepare repair SQL for manual execution by default. Execute writes or maintenance only when the user has authorized that action in the identified environment; existing authorization remains valid. This does not prevent local code changes or permitted read-only diagnosis.

Deliver the evidence, proposed SQL/XML or code, expected effect, and relevant lock/recovery implications. When the environment is unavailable, identify the smallest missing diagnostic rather than asserting a root cause.

Use Adobe's [schema and patch documentation](https://developer.adobe.com/commerce/php/development/components/declarative-schema/patches) for application-managed migrations.
