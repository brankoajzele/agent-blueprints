# Optional workstation performance profile

## Development performance defaults

This optional profile comes from a 32 GB workstation (approximately 27 GiB usable by Linux). Apply it when the user requests this tuning or the available capacity and intended concurrency support it. Inspect existing settings and host capacity first; do not assume these allocations suit every machine. For a small number of simultaneously active projects, the starting settings are:

| Setting | Starting value |
| --- | --- |
| InnoDB buffer pool | 4 GiB per active project |
| MySQL redo capacity | 512 MiB |
| MariaDB InnoDB log file size | 512 MiB, where supported |
| OpenSearch JVM heap | `-Xms2g -Xmx2g` |
| PHP-FPM maximum children | 8 per PHP container |
| OPcache memory | 512 MiB; preserve timestamp checking for development |

Persist settings in the project's root `.warden/` files and mounts, keeping MySQL and MariaDB options distinct. Preserve existing overrides. Budget memory across all active projects and desktop applications; do not allocate these amounts to unlimited concurrent environments. Do not relax database durability as a permanent performance shortcut.

Validate the merged Compose configuration and service-specific settings. Restart only the affected running project services. Do not start unrelated stopped projects.
