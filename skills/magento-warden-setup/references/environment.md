# Environment and local Magento configuration

## Inspect and configure Warden

- Work from `~/projects/<project>`, not `src/`, for Warden commands.
- If root `.env` is missing, copy `src/.env`, otherwise `src/.env.sample`, when available. Review the contents first. If neither exists, initialize a Magento 2 Warden environment.
- If root `.warden/` is missing, copy `src/.warden/` when available. Preserve existing root overrides and merge only the required changes.
- Set `WARDEN_ENV_NAME=<project>`, `WARDEN_ENV_TYPE=magento2`, `WARDEN_WEB_ROOT=/src`, `TRAEFIK_DOMAIN=<project>.test`, and `TRAEFIK_SUBDOMAIN=app`.
- Check copied bind-mount paths: they must resolve from the project root. Keep source-code-specific mounts pointed at `./src/...` and Warden configuration mounts at `./.warden/...`.
- Determine compatible PHP, database, search, and other service versions from the locked Magento version, repository instructions, and existing configuration. Do not blindly copy another project's service versions.
- Validate `warden env config`; verify `src/` mounts at `/var/www/html`.
- Start required global Warden services and the project. Sign a local certificate using `warden sign-certificate <project>.test`.
- Add missing local hostnames to `/etc/hosts`, normally mapped to `127.0.0.1`. Preserve unrelated entries. Obtain system authentication when required.
- Verify HTTPS trust in command-line clients and Chrome. Preserve existing trust stores.

Run all PHP, Composer, and Magento commands **inside Warden containers**, never with host PHP. For example, from the project root:

```bash
warden env exec -T php-fpm composer install --no-interaction --no-progress
warden env exec -T php-fpm php bin/magento setup:db:status
```

Use the lockfile and the repository's Composer scripts and patches. Inspect and address failures; do not bypass failed patches or change dependency versions merely to get installation to finish. Reuse existing Composer authentication without printing its contents.

## Build the local Magento environment

- Create or adapt untracked `src/app/etc/env.php` to use the project's local database and enabled local services. Do not carry over staging database, cache, queue, or search endpoints.
- Preserve the Magento encryption key associated with the dump when available. Look for an authorized existing environment configuration or supplied key. If it is missing, explain that encrypted settings cannot be preserved and request the key when required; do not silently substitute an unrelated project's key.
- Match the installed search engine, including ElasticSuite when the repository uses it. Configure its server, credentials, protocol, and index prefix for the local search container.
- Keep changes specific to the machine in ignored environment/configuration files rather than modifying tracked production configuration.
- Keep general staging cron jobs, external synchronization, live queue consumers, and outbound business automation off locally. Route local email through Warden's mail service. Enable local indexing cron when appropriate; persist its crontab across container recreation.
- Use developer mode unless the user asks otherwise. Configure local HTTPS/offloader handling and usable local admin authentication. Remove staging-only reCAPTCHA bindings locally when they prevent local use; use the installed version's supported disabled value rather than assuming an empty string works.
- Set the local admin front name to `admin_dev` in untracked `src/app/etc/env.php`, so the admin URL is always `https://app.<project>.test/admin_dev`. Use the Magento CLI inside Warden:

  ```bash
  warden env exec -T php-fpm php bin/magento setup:config:set --backend-frontname=admin_dev --no-interaction
  ```

- For **every setup**, create an additional local administrator through the Magento CLI with username **`developer`**, password **`developer123`**, and interface locale **`en_GB`** (British English). Preserve other imported admin users. If `developer` already exists, reconcile that account with these local credentials and locale instead of attempting a duplicate. These are the user's requested standard local-development credentials, not credentials to apply to staging or production.

  ```bash
  # Replace <project> with the actual project name.
  warden env exec -T php-fpm php bin/magento admin:user:create \
      --admin-user=developer --admin-password=developer123 \
      --admin-email="developer@<project>.test" \
      --admin-firstname=Developer --admin-lastname=Local
  ```

  The standard account-creation command may not support a locale option. Set the interface locale through Magento's user model using PHP inside the container; do not invent an unsupported CLI flag:

  ```bash
  warden env exec -T php-fpm php -r '
  require "app/bootstrap.php";
  $bootstrap = \Magento\Framework\App\Bootstrap::create(BP, $_SERVER);
  $om = $bootstrap->getObjectManager();
  $om->get(\Magento\Framework\App\State::class)->setAreaCode("adminhtml");
  $user = $om->create(\Magento\User\Model\User::class)->loadByUsername("developer");
  if (!$user->getId()) { throw new \RuntimeException("Developer admin was not created"); }
  $user->setInterfaceLocale("en_GB")->save();
  '
  ```

  Include `en_GB` in admin static-content deployment even if the storefront uses different locales. Verify an actual login as `developer` at `/admin_dev` and confirm the stored interface locale is `en_GB`. Report the standard local credentials and admin URL at handoff.
