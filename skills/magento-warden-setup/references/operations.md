# Everyday operations and troubleshooting

## Everyday commands

Run these from the **project root above `src/`**. Use the project's Warden environment rather than guessing a container name from another project.

```bash
# Start or reconcile this project's containers; start global services if needed.
warden svc up
warden env up -d

# Inspect this project and its recent service logs.
warden env ps
warden env logs --tail=100 php-fpm nginx db opensearch

# Enter its PHP container; PHP/Composer/Magento commands run here.
warden shell

# Stop this project when finished to free its RAM.
warden env stop
```

Global Warden services are shared. Do not stop them merely to stop one project, and do not remove project volumes as a routine troubleshooting step. `warden env stop` preserves the containers and database volumes; `warden env up -d` also applies pending configuration changes when starting again.

```bash
# Check schema and indexer state without initiating a reindex.
warden env exec -T php-fpm php bin/magento setup:db:status
warden env exec -T php-fpm php bin/magento indexer:status

# Clean Magento caches while retaining the required local enable/disable state.
warden env exec -T php-fpm php bin/magento cache:clean
warden env exec -T php-fpm php bin/magento cache:status

# Restore the required state after a tool or command changes it.
warden env exec -T php-fpm php bin/magento cache:enable
warden env exec -T php-fpm php bin/magento cache:disable full_page block_html
warden env exec -T php-fpm php bin/magento cache:status

# Interactive access to this project's local database.
warden db connect
```

`warden shell` opens an interactive session: exit it before executing host-side Warden commands. Do not use host PHP when a shell session ends. Cache cleaning does not authorize reindexing, dependency upgrades, general cron execution, image regeneration, or sending test emails/orders to external services.

For code/configuration changes, restart or recreate only the affected services. Validate nginx/PHP/database configuration before applying it. Review repository instructions before running `setup:upgrade`, configuration imports, or theme build tools; these are not universal fixes for every slow page.

## Focused troubleshooting

| Symptom | Check and response |
| --- | --- |
| Hostname fails or certificate warning | Check the exact `/etc/hosts` mapping, certificate SAN coverage, Warden router rules, global Traefik status, and browser trust. Do not mask failures by disabling TLS verification as the final solution. |
| Wrong store or redirect to staging | Inspect scoped URL overrides, store-code behavior, `MAGE_RUN_TYPE`/`MAGE_RUN_CODE`, existing routing, and cookies. Test a fresh session and the real switcher. Changing hosts alone cannot select the Magento scope. |
| 502 after service recreation | Check PHP readiness, service logs, and nginx's upstream resolution. Validate then reload/restart the affected proxy if it retains a stale upstream address. |
| Cached pre-install 404 | Compare the direct local backend with the public response and inspect Varnish headers. Clear the affected project's stale proxy cache; do not restart unrelated projects or delete database volumes. |
| Configuration-file-changed error | Synchronize the intentionally changed local configuration using the appropriate `app:config:import` or `setup:upgrade`, then verify the required cache state again. |
| Category/search fails with missing indexes | Report deferred reindexing and provide the user-run command. Do not start a lengthy full or targeted rebuild automatically. |
| Missing images/banners | Expected when media was not supplied. Keep local media URLs; do not download/proxy staging media, crawl image URLs, or regenerate the full image cache. |
| Admin login fails | Verify `/admin_dev`, the local account/password and active state, `en_GB`, HTTPS/cookie scope, and local CAPTCHA/2FA settings. Preserve other imported accounts. |
| Blank or broken styles/scripts | Check actual theme/locale deployment, HTTP responses for local assets, file permissions, bundling/minification, and static URL/sign settings. Do not run a production-mode conversion as a generic fix. |
| Slow requests or high CPU | Identify the process and whether the request is uncached, compiling, querying, or searching. Check configured resources and logs before changing limits. Full-page and block HTML caches are intentionally disabled for this workflow. |
| Import fails on collation/definers | Verify the database distribution/version and the installed Warden import filters. Apply the smallest compatible transformation to the import stream and retain the untouched dump. |

Inspect logs with care: do not paste secrets, full environment files, decrypted dump content, or customer records into progress updates or reports. Capture only the relevant error and the evidence needed to explain the fix.
