---
name: magento-operations
description: Diagnose Magento infrastructure, caching, queue, Cron, and deployment problems, or implement requested operational changes. Use for runtime operations rather than ordinary module development.
---

# Magento operations

## Establish the environment

Identify the deployment platform, affected environment, installed service versions, and existing runbooks. Check the release-specific support matrix before proposing service upgrades. Preserve the current topology unless evidence or the request calls for a change.

## Diagnose before tuning

- Locate the bottleneck using timings, correlated logs, and resource metrics across CDN/proxy, PHP-FPM, database, cache, and search. Compare against a known baseline when available.
- For caching, distinguish Magento application cache, full-page cache, sessions, and edge caching. Inspect hit rate, cacheability, invalidation, memory, and eviction evidence before changing TTLs, backends, or policies. Logical Redis databases share instance memory and eviction policy.
- For Cron and queues, inspect schedule/backlog, failures, locks, consumer supervision, and retry behavior. Confirm the actual queue backend and runner before using backend-specific commands.
- Derive worker counts, memory limits, and batch sizes from workload and available capacity. Shared media, centralized logs, and additional nodes are topology decisions, not prerequisites for every store.

## Make operational changes

Use existing deployment automation and configuration ownership. Specify target services and expected effects for commands; avoid broad cache flushes, service restarts, or reindexing as diagnostic defaults. Live mutations must fit the user's authorization and environment runbook.

For deployments, account for code, generated assets, schema/data patches, and running workers. A symlink switch does not make database upgrades atomic or guarantee zero downtime. Use backward-compatible migrations where possible and distinguish code rollback from data recovery. Verify permissions for the actual filesystem owner and web runtime; protect secrets and public document-root boundaries.

Validate changed behavior with relevant health and business checks. Report evidence, changes, verification, and any remaining operational step.

Consult Adobe's [system requirements](https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/system-requirements) and [filesystem permissions](https://experienceleague.adobe.com/en/docs/commerce-operations/configuration-guide/deployment/file-system-permissions) for the target deployment.
