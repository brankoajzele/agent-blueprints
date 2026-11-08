---
name: magento-warden-setup
description: Set up or resume a local Magento Warden environment from an existing repository in src/ and a database dump in the project root, including encrypted imports, multi-store domains, local admin access, and verification. Use for local environment bootstrap or restore rather than production deployments or general Magento runtime diagnosis.
---

# Magento Warden setup

Turn a cloned Magento repository and supplied database dump into a usable local environment. Work autonomously through setup and verification; ask only for missing essentials that cannot be discovered, such as the dump passphrase, repository credentials, encryption key, or the intended dump when several are plausible.

## Local workflow and constraints

- Keep Warden `.env` and `.warden/` in the project root, with the repository in `src/` and `WARDEN_WEB_ROOT=/src`. Run Warden commands from that root.
- Read applicable project instructions and inspect existing state. Reuse valid configuration and completed work; preserve user changes and imported users. Do not assume an existing database can be replaced merely because a dump exists.
- Execute PHP, Composer, and Magento commands inside Warden containers. Use the lockfile and repository patch scripts; do not upgrade dependencies to work around setup failures.
- Infer `app.<project>.test` from the root directory unless the user supplies another mapping. Inspect every active website and store view rather than configuring only the default scope.
- **Skip reindexing unless explicitly requested**, including targeted rebuilds and indirect full rebuilds through cron. Report missing indexes and any checks they block; give the manual command instead.
- Do not download, proxy, or crawl staging media unless explicitly requested. Missing media is an expected limitation when only a dump was supplied.
- Keep general staging cron, live queue consumers, and external business automation off locally. Enable appropriate indexing cron only if it will not trigger a deferred rebuild.
- Keep secrets out of tracked source, logs, and reports. A remote export requires authorization for that server; an existing local dump does not imply it.

## Agreed local-development profile

These are deliberate defaults for this setup workflow, subject to explicit user/project overrides. Apply them only to the identified local environment:

- Admin front name: `admin_dev`.
- Additional CLI-created administrator: `developer` / `developer123`; interface locale `en_GB`. Preserve other users; reconcile this account if it already exists.
- Developer mode. All Magento caches enabled **except `full_page` and `block_html`, which must be disabled** at completion.
- No automatic full reindexing and no staging media transfer.

Read [environment.md](references/environment.md) for the admin creation/locale commands and local service configuration. Read [verification-and-handoff.md](references/verification-and-handoff.md) for cache commands and exact completion requirements. Workstation-specific RAM tuning is optional; read [performance.md](references/performance.md) only when sizing or changing resources.

## Setup or resume

1. Inspect project instructions, Git status, source/lockfile, root overrides, supplied dumps, running services, and an existing `.warden/setup-report.txt`. Treat the report as context, then verify actual state.
2. Read [environment.md](references/environment.md). Copy source `.env` or `.env.sample` and `.warden/` only when root equivalents are missing, adapt mounts, choose compatible services, and validate Compose. Configure hosts and HTTPS through the available system authentication flow.
3. Read [database-imports.md](references/database-imports.md) for the supplied format. Stream encrypted gzip dumps using a protected passphrase descriptor, `pigz` when available, and `pipefail`; retain the original. Apply collation/definer changes only when needed. Back up an existing local database before an authorized replacement.
4. Create local `src/app/etc/env.php` using the dump's encryption key when available and the local database/cache/queue/search endpoints. Configure ElasticSuite when installed. Apply the local-development profile and deactivate staging-only integrations locally.
5. Read [domains-and-stores.md](references/domains-and-stores.md). Map every active website/store to local URLs; reconcile default/website/store overrides, cookies, certificate coverage, proxy rules, and Magento scope selection. Prefer nginx/FastCGI routing. Disclose and preserve reversal instructions for any necessary local `index.php` fallback.
6. Read [verification-and-handoff.md](references/verification-and-handoff.md). Synchronize changed configuration/schema as required, deploy actual themes/locales, enforce cache state, and verify what can be checked without launching a reindex. If errors arise, consult the relevant part of [operations.md](references/operations.md).

For a later reimport, use the replacement procedure in [database-imports.md](references/database-imports.md), then reapply store URLs, local services, admin settings, and cache state. Do not assume they survive a new dump.

## Delivery

Save or update the private, non-secret operational record `.warden/setup-report.txt` outside `src/`. Report admin URL/username/password/locale and a quick-access table for **every active store view**, with homepage, one active product, category, and contact URL. Discover real store-scoped links; mark unavailable or unverified links with the reason rather than inventing them.

Explain exact hosts entries and how each hostname or path selects Magento's website/store. Include routing/configuration paths, any temporary entry-point edit and how to revert it, dump/report locations, final cache and cron state, omitted media, and the skipped-reindex command. Do not claim blocked category/search checks passed. Finish only after commands you started have completed; intentionally deferred indexing is a documented handoff.
