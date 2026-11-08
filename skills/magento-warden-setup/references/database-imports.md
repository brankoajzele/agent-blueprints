# Database imports and replacement

## Import the supplied database

Identify the dump format and preserve the original file. Do not fetch staging media or other large assets unless the user explicitly requests it. Do not connect to a remote server unless the user requests a remote export or the current task otherwise authorizes it.

For a GPG-encrypted gzip dump, use the streaming path below in **Bash**, from the project root. It avoids an intermediate plaintext SQL file and `pv`. Obtain the passphrase from the user or an explicitly supplied secret source. Never put real passphrases in this guide, tracked files, command arguments, or progress output. Do not enable shell tracing.

```bash
(
    set -o pipefail
    read -r -s -p 'Dump passphrase: ' DUMP_PASSPHRASE
    printf '\n'
    trap 'unset DUMP_PASSPHRASE' EXIT

    gpg --batch --yes --quiet --pinentry-mode loopback \
        --passphrase-fd 3 --decrypt ./magento.gpg \
        3<<<"$DUMP_PASSPHRASE" \
      | pigz -dc \
      | LC_ALL=C sed 's/utf8mb4_0900_ai_ci/utf8mb4_general_ci/g' \
      | warden db import
)
```

Replace `./magento.gpg` with the actual filename. In an agent session without an interactive terminal, adapt the passphrase delivery to a protected file descriptor; do not hardcode it into a saved script. Use `gzip -dc` if `pigz` is unavailable, or install `pigz` through the normal system permission flow when worthwhile. Confirm the decrypted payload is gzip; encrypted plain SQL needs no decompression stage.

The collation replacement is a compatibility option, mainly for importing MySQL 8 dumps into MariaDB. Inspect the source and target first; omit it when the original collation is supported and preserving it is appropriate. Do not apply unrelated blanket text replacements to the dump.

For unencrypted dumps:

```bash
# gzip-compressed SQL
set -o pipefail
pigz -dc ./magento.sql.gz | warden db import

# Plain SQL
warden db import < ./magento.sql
```

Check the full pipeline's exit status. Treat decryption, decompression, SQL errors, and partial imports as failures. Inspect Warden's installed import implementation before adding duplicate DEFINER/GTID filters; it may already handle them. Resolve any remaining definer privileges locally without granting broad privileges to Magento's application account. Do not change the original backup.

## Reimporting or replacing an existing local database

A fresh environment import and replacing a used local database are different operations. Before replacing an existing local database:

1. Identify the intended project, target local database, source dump, and current local changes. A file's presence alone does not authorize destroying existing work. If the current request does not authorize replacement, finish the assessment and obtain the user's decision before replacing it.
2. Preserve the current `env.php`, local overrides, routing changes, and setup report in a private backup. Record the existing encryption-key source without printing the key.
3. Make a complete backup of the current local database and verify the backup pipeline succeeds. Protect it because it may contain customer data. For example, when `pigz` is available:

   ```bash
   (
       set -euo pipefail
       umask 077
       mkdir -p ./backups
       backup_file="./backups/local-before-reimport-$(date +%Y%m%d-%H%M%S).sql.gz"
       test ! -e "$backup_file"
       warden db dump | pigz > "$backup_file"
       gzip -t "$backup_file"
       printf 'Backup saved: %s\n' "$backup_file"
   )
   ```

   A compression integrity check does not by itself prove SQL consistency; check the dump command and pipeline exit status as well. If backup creation fails, do not proceed with replacement.
4. Prevent local requests and scheduled writers during replacement using an appropriate maintenance window. Record any running local cron/consumer configuration so it can be restored deliberately. Do not touch another project's services.
5. Import using the supplied dump's correct decryption/decompression pipeline. An import that stops midway leaves a partial database; do not present that as a usable restored environment. Avoid blindly importing over a partial schema or assuming removed tables disappear: inspect the dump's table-drop behavior and use a deliberate, authorized local replacement procedure.
6. Reconcile the imported dump's encryption key, website/store mappings, URLs, cookie settings, local services, external integrations, `/admin_dev`, and the `developer` account with locale `en_GB` again. A new dump can replace all of these database settings.
7. Run required upgrades/configuration imports, clear caches, restore the required cache state, and verify each store and admin access. **Do not automatically reindex**, including through resumed cron, as a shortcut to make verification pass.
8. Restore only appropriate local scheduling, update the setup report, and retain the previous dump/configuration backups until the user chooses to remove them.

If rollback is required, assess and restore the matching database/configuration pair, including its encryption key. Do not assume a database dump alone reverses code changes, patches, uploaded files, or service-version changes.
