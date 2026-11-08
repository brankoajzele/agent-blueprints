# Verification, handoff, and persistent setup report

## Complete and verify

1. Run required schema/configuration upgrades and deploy static assets for the actual frontend/admin themes and enabled locales.
2. **Skip full reindexing by default.** Projects can be massive and a full rebuild can take hours. Do not run `bin/magento indexer:reindex` without indexer IDs, launch the equivalent through another command, or start a background full rebuild unless the user explicitly requests it. Inspect `indexer:status` and search-cluster health instead. Do not reset indexers or invalidate existing indexes merely to force a rebuild. Do not trigger a full rebuild indirectly through cron as a workaround; if scheduled indexing would immediately rebuild an invalid index, defer that indexing cron and explain why. Targeted reindexing also requires an explicit user request.

   Tell the user that reindexing was intentionally skipped, report any invalid/missing indexes, and provide the manual command for when they are ready:

   ```bash
   # From the project root; may take hours on large catalogs.
   warden env exec -T php-fpm php bin/magento indexer:reindex
   ```

   Continue independent setup and verification. If missing indexes prevent category/search checks, record those checks as blocked pending the user-run reindex; do not claim they passed. A deferred reindex is an expected handoff, not a reason to launch it automatically.
3. Clear Magento caches and stale pre-install responses in Varnish. Leave **all Magento cache types enabled except `full_page` and `block_html`, which must be disabled**. Run these commands inside Warden, then verify the final state with `cache:status`:

   ```bash
   warden env exec -T php-fpm php bin/magento cache:enable
   warden env exec -T php-fpm php bin/magento cache:disable full_page block_html
   warden env exec -T php-fpm php bin/magento cache:status
   ```

   Check persistent indexing cron if enabled. This cache state is required at completion, including after any later setup or verification commands.
4. Verify trusted HTTPS, the homepage, a category, a product, search, local scripts/styles, and a real local admin dashboard login as `developer` at `/admin_dev` with interface locale `en_GB`. Check every active store view's URL and language switching; ensure no accidental staging redirects or stale cookie domains remain.
5. Missing media is expected when it was not supplied. Report it without downloading media or attempting full image regeneration.
6. Check repository status, preserve existing changes, and remove temporary secret files. Keep backups and local credentials private and outside tracked source files.
7. Save or update `.warden/setup-report.txt` as described under Persistent local setup report and provide the full handoff described below. Distinguish verified results from assumptions. Do not report completion while imports or other setup commands you started are still running. Explicitly identify deferred reindexing and any verification it prevents; setup may be handed back with those documented limitations.

## Required handoff and quick-access links

Give the user a self-contained final response containing:

- The full admin login URL, normally `https://app.<project>.test/admin_dev`, username `developer`, password `developer123`, and admin interface locale `en_GB`.
- A table covering **every active website/store view**, with website code, store-view code, locale, homepage URL, one active product page URL, one category page URL, and the contact page URL for that store.
- The exact `/etc/hosts` entries added and an explanation of how each host or store-code path selects its Magento website/store view.
- The routing implementation and configuration paths. Explicitly disclose any temporary `index.php` change and its reversal instructions.
- The saved `.warden/setup-report.txt` location, database backup location, import result, cache state (`full_page` and `block_html` disabled; other cache types enabled), and whether indexing cron or other automation is enabled.
- A clear statement that full reindexing was skipped, the manual command to run it, and any checks blocked by absent indexes. State when media was intentionally not copied.

Use this quick-access layout, with clickable full local URLs rather than paths alone:

| Website / store view | Locale | Homepage | Active product | Category | Contact |
| --- | --- | --- | --- | --- | --- |
| Actual codes | Actual locale | Verified URL | Verified URL | Verified URL | Verified URL |

Discover real product and category URLs from the imported store-scoped URL rewrites and catalog configuration. Choose an enabled, storefront-visible product assigned to the website and an active category available in that store. Verify the chosen product is actually accessible there; do not reuse another store's URL without checking it. Use a real enabled contact route or project-specific contact CMS page and verify it rather than assuming every store exposes `/contact/`.

Do not invent links to fill the table. If no suitable product/category exists, contacts are disabled, or a check is blocked by deferred indexing, mark that cell unavailable or unverified and give the specific reason. Still list every active storefront homepage and any discovered intended links, clearly labeling their verification status. Note intentionally disabled stores separately if relevant. The final response must give the user all quick-access links without requiring them to read earlier progress messages.

## Persistent local setup report

Create and maintain `<project-root>/.warden/setup-report.txt` for each setup. This is a local operational record outside the cloned `src/` repository, not a replacement for the full final response. Read it at the start of subsequent work and reconcile its contents with the actual machine state before relying on it.

Record:

- Project root, source path, setup date, source commit, and relevant pre-existing local changes.
- Warden environment name and actual PHP, Magento, database, search, and other service versions.
- Website/store-view codes, locales, local hostnames and paths, homepage/product/category/contact links, and their verification status.
- Admin URL and username, plus the location of any private credentials file. Do not duplicate passwords in this report.
- Exact hosts entries, certificate domains, routing implementation, and local configuration file paths.
- Imported dump filename, size, checksum when available, import date/result, and any import transformations. Record the source environment only when known.
- Whether the Magento encryption key was preserved, without recording the key itself.
- Final cache status, cron groups enabled, queue-consumer policy, disabled local integrations, media availability, and performance overrides.
- Commands or checks completed, deferred reindexing, blocked checks, and actionable follow-up work.
- Configuration backup locations and how to reverse any local entry-point patch.

Keep this report free of dump passphrases, private keys, Composer tokens, database passwords, and customer/order records. Store actual private credentials separately with restrictive permissions. Ensure local artifacts cannot accidentally be committed if the project root is itself a Git repository.

Do not treat the report as an executable script. Recheck paths, running containers, and database identity before any import or destructive operation. Update the report whenever later work changes the setup.
