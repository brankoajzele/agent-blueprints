---
name: magento-backend-development
description: Implement or fix Magento modules, PHP services, APIs, plugins, observers, and background jobs. Use for backend implementation rather than standalone architecture or review requests.
---

# Magento backend development

## Establish the change

Read the affected module and its callers. Infer edition, Magento/PHP versions, module ownership, and execution area from the project before asking for missing context. Follow existing conventions and keep the implementation proportional to the request.

## Implementation decisions

- Extend through supported APIs, plugins, observers, or local modules; keep installed `vendor/` files unchanged. Choose the extension point by the required behavior, not a fixed preference order. Use an `around` plugin when control over the original invocation is necessary.
- Inject dependencies rather than calling `ObjectManager` directly. Add service contracts for public module boundaries or integrations; an internal helper does not automatically need an `Api/` interface. Keep entry points thin when logic needs reuse or independent testing.
- Use repositories for domain operations and resource models/collections for persistence and bulk work. Parameterized adapter queries belong in the persistence layer when higher-level APIs cannot express the operation efficiently. Preserve domain side effects and avoid per-entity loads in loops.
- Use `db_schema.xml` for supported schema changes and data patches for one-time migrations. Check existing schema ownership and patch dependencies.
- Resolve configuration at the intended default, website, or store scope. Add a configuration wrapper, feature toggle, or custom log channel when it serves a concrete need. Use the existing PSR-3 logger and exclude secrets and personal data.
- For long-running or retryable work, choose synchronous execution, Cron, or a queue based on latency and operational needs. Make retries idempotent and bound batch size. Add a CLI entry point when manual execution is useful; share the underlying logic. Use store emulation only when required, and restore it in `finally`.
- Match supported PHP features and inherited signatures. Do not force new types onto framework overrides that cannot accept them.

## Verify and deliver

Run the project's relevant checks and test changed business behavior. Compile DI when wiring changes warrant it and the environment supports it; clean only affected caches. Update existing documentation when configuration, interfaces, or operation changes. Report the behavior, validation, and any checks that could not run.

Consult Adobe's [schema and data patch guidance](https://developer.adobe.com/commerce/php/development/components/declarative-schema/patches) when changing persistence setup.
