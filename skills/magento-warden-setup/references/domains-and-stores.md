# Store URLs and multi-domain routing

## Reconcile URLs and cookies for every store

Inspect `store_website`, `store_group`, `store`, and relevant `core_config_data` rows. Build a mapping for **every active website and store view** before applying updates, accounting for default, website, and store scopes plus overrides in `env.php` and `config.php`.

Review at least:

- `web/unsecure/base_url` and `web/secure/base_url`.
- Base link, static, and media URLs under both secure and unsecure configuration.
- `web/cookie/cookie_domain`, cookie path, and secure-cookie settings.
- `web/url/use_store`, redirect-to-base behavior, and secure frontend/admin settings.
- Language-switcher URLs and existing host-based website routing.

Use a shared local hostname with existing store-code paths where that matches the project, for example `https://app.<project>.test/default/` and `/default_en/`. When websites need distinct hosts, configure Warden/nginx Magento website routing, host entries, and certificate coverage for each hostname. DNS and URL edits alone do not select a Magento website. Preserve the intended website/store structure and language codes.

### Multiple local domains and Magento scope routing

For a multi-website or multi-store database, inspect the real website/store codes and existing routing before choosing local domains. Preserve its intended structure. Prefer subdomains covered by the project's Warden router and wildcard certificate, for example `app.<project>.test`, `en.<project>.test`, and `b2b.<project>.test`. Separate `.test` domains are also possible, but require explicit proxy router rules and matching certificates; do not assume Warden routes them automatically.

Create an explicit mapping containing hostname, website code, store-view code, locale, and whether access uses a domain, store-code path, or both. Add every local hostname to `/etc/hosts`, for example:

```text
127.0.0.1 app.example.test en.example.test b2b.example.test
```

Explain that `/etc/hosts` maps each hostname to this machine; Warden/Traefik selects the project, nginx/PHP selects the Magento scope, and scoped base URLs determine generated links. All of these layers must agree.

Prefer project-local nginx/FastCGI configuration in root `.warden/` to pass Magento's bootstrap parameters:

- `MAGE_RUN_TYPE=website` with `MAGE_RUN_CODE=<website code>` selects a website and lets Magento resolve a store within it.
- `MAGE_RUN_TYPE=store` with `MAGE_RUN_CODE=<store-view code>` selects a particular store view.
- Use actual codes from the imported database, not numeric IDs or display names.

Inspect `nginx -T` inside the container before adding configuration. Put `map` directives in the nginx `http` context and FastCGI parameters in the appropriate PHP location; do not paste an `http`-only directive into a server-level include or add conflicting duplicate parameters. Account for existing custom routing. Validate using `nginx -t` before reloading, then test every mapped hostname.

If the installed Warden/nginx template cannot support the required mapping cleanly, a temporary local front-controller change is an allowed fallback. Inspect the real entry point (normally `src/pub/index.php`) and set `MAGE_RUN_TYPE` and `MAGE_RUN_CODE` in the parameters passed to `Bootstrap::create(...)`, before the bootstrap is created. Use an explicit allowlist of the exact local hostnames; leave unmatched hostnames' existing behavior intact. Do not change the bootstrap after it has already been instantiated or modify a different unused index file.

Keep that fallback minimal and local. Preserve the original file or exact reverse patch, do not commit the local routing change, and explain precisely which file changed, how it selects each scope, and how to revert it. Check whether Composer or later deployment commands replace the entry point, and verify routing again afterward. Prefer the nginx approach when feasible so tracked Magento entry points remain unchanged.

Verify each store both in a fresh session and through its actual language/store switcher. Check correct language, currency, generated URLs, cookie scope/path, and absence of staging redirects. Do not force unrelated websites to share a cookie domain merely because their local hosts share a suffix.

Remove stale staging cookie domains and URL overrides at **all applicable scopes**. Preserve valid inherited placeholders such as `{{secure_base_url}}` where appropriate. Keep media URLs local even when media has not been copied; do not compensate by fetching or proxying remote media without authorization.

Do not blindly replace every occurrence of a staging hostname throughout the database: content, order history, integration endpoints, and serialized values may require different treatment. Make targeted configuration changes and verify the resulting effective store URLs.

After editing locked configuration, run the required `app:config:import` or `setup:upgrade` so Magento does not reject requests because the configuration hash has changed.
