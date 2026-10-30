# Magento local development

This project runs in Warden-managed Docker containers. Run Magento CLI, Composer, PHPUnit, and all other project PHP commands inside the PHP container. Do not use host PHP, including host execution of scripts with PHP shebangs.

## Discover the environment

Run host-side Warden commands from the project directory containing `.env`. Inspect the relevant values in `.env`, any supported local overrides, and `.warden/` before choosing commands:

- `WARDEN_ENV_NAME` and `WARDEN_ENV_TYPE` identify the environment.
- `WARDEN_WEB_ROOT` determines which host directory maps to the application root; do not assume source lives at the repository root or in `src/`.
- `TRAEFIK_DOMAIN` and `TRAEFIK_SUBDOMAIN` define default routing. The domain need not match `WARDEN_ENV_NAME`; Magento store configuration may use additional URLs.
- Enabled services, versions, and Compose overrides determine the available database, search, cache, and queue backends.

Use `warden env ps` for service status and `warden env config` when merged configuration is needed. Inspect configuration selectively; keep credentials and secret values out of reports.

Prefer Warden service commands over constructing container names. Warden resolves the project without manually parsing `.env`, exporting persistent variables, or relying on a container-name suffix.

## PHP and Magento commands

Use `warden shell` for interactive PHP-container access. For scripted commands, disable TTY allocation:

```bash
warden env exec -T php-fpm php --version
warden env exec -T php-fpm composer --version
warden env exec -T php-fpm php bin/magento --version
warden env exec -T php-fpm php bin/magento indexer:status
```

These examples assume the standard application working directory, `/var/www/html`. Check the actual mount and working directory if a file is missing; use Compose's `-w` option when an override requires it. Do not compensate by switching to host PHP.

Run project test and build commands with the configured dependencies. PHPUnit runs in the PHP container. Browser/Node tooling may run on the host or in its configured container; select the base URL and artifact paths for that execution location. Integration tests require their own configured test database.

If direct Docker access is needed, resolve the service ID rather than guessing a name:

```bash
PHP_CONTAINER="$(warden env ps -q php-fpm)"
[ -n "$PHP_CONTAINER" ] || { echo "PHP service not found" >&2; exit 1; }
docker exec "$PHP_CONTAINER" php bin/magento --version
```

Use `-it` only for interactive Docker commands. If the environment is unavailable, report that limitation; do not initialize a replacement environment or change service versions as an automatic workaround.

## Database and search access

For an interactive database session, use `warden db connect`. For scripted diagnostics with standard Warden database variables:

```bash
warden env exec -T db sh -c '
  MYSQL_PWD="$MYSQL_PASSWORD" mysql \
    --user="$MYSQL_USER" --database="$MYSQL_DATABASE" \
    --batch --skip-column-names --execute="SELECT VERSION(), DATABASE();"
'
```

Use the installed database client and actual credentials if the service is customized. Verify Magento's configured connection and table prefix before application queries. Database imports, resets, and repairs are separate operations from establishing access.

Reach search through the configured provider/service on Docker's internal network. For a standard enabled OpenSearch service:

```bash
warden env exec -T php-fpm curl --fail --silent --show-error \
  'http://opensearch:9200/_cluster/health?pretty'
```

Use `elasticsearch` only when that is the configured service. Follow actual protocol, port, and authentication settings. A host browser or host-side script cannot generally use Docker-internal service names.

## Storefront and browser checks

Use the actual storefront/store URL. This HTTP smoke check derives the default URL from the PHP container's routing variables:

```bash
warden env exec -T php-fpm bash -lc '
  : "${TRAEFIK_DOMAIN:?TRAEFIK_DOMAIN is required}"
  base_url="https://${TRAEFIK_SUBDOMAIN:-app}.${TRAEFIK_DOMAIN}"
  curl --fail --silent --show-error --location \
    --output /dev/null --write-out "%{http_code}\n" "$base_url/"
'
```

Prefer trusting Warden's local certificate authority. If local certificate validation blocks a check, scope any cURL `--insecure` or Playwright `ignoreHTTPSErrors` exception to that local test. For DNS failures, diagnose resolution from the command's execution location. A direct request to `nginx` must preserve the intended Host header and scheme behavior; it does not verify the public local routing path.

For session-dependent HTTP checks, reuse a cookie jar with `--cookie` and `--cookie-jar`. Store local artifacts under Magento `var/smoke/` or the project's test output directory, and exclude cookies, saved authentication, and generated evidence from Git. Resolve the corresponding host path from the actual mount.

Use Playwright for customer interactions and JavaScript-dependent assertions. Prepare only the scenario data needed, clean up owned records, and keep browser fixtures separate from integration-test database setup. HTTP success alone does not establish that a customer journey passed.

## Logs and targeted maintenance

```bash
warden env exec -T php-fpm tail -n 120 var/log/system.log var/log/exception.log
```

Inspect module-specific logs and configuration paths from the code being investigated. Record original scoped settings before temporarily enabling logging and restore them afterward.

Clean only cache types affected by a change. Run compilation, schema upgrades, reindexing, and container lifecycle commands when the task requires them. Keep volume deletion, database replacement, and blanket cache-backend flushing out of routine troubleshooting.
