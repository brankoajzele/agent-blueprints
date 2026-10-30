---
name: magento-upgrade-assessment
description: Assess a Magento Open Source or Adobe Commerce project against a target release, identify compatibility blockers, and plan upgrade validation. Use for upgrade readiness and scoping before implementation.
---

# Magento upgrade assessment

## Establish the baseline and target

Read project instructions, `composer.json`, `composer.lock`, deployment configuration, and existing patches. Identify edition, exact current release including patch suffix, hosting model, storefront, extensions, and custom modules. Distinguish locked dependencies from the deployed runtime; mark unavailable environment facts as unknown.

Confirm the exact target release. If asked to recommend one, check current release status, support lifecycle, and project constraints; distinguish stable releases from previews. Treat an edition or hosting-model migration as additional scope.

## Assess compatibility

- Compare current and target PHP, Composer, database, search, cache, and queue requirements against the applicable hosting support matrix. Identify required service transitions and whether an intermediate upgrade step is documented or necessary.
- Inspect Composer constraints, repositories, replacements, and extension dependencies. Use dependency diagnostics to explain conflicts. Perform any target dependency resolution in an isolated working copy, accounting for Composer plugins/scripts; preserve the project's manifest, lockfile, and installed packages during assessment.
- Check extension and storefront compatibility against vendor documentation for the exact target. A permissive Composer constraint or successful dependency resolution does not prove runtime compatibility. Record unsupported or unverified packages explicitly.
- Review target release notes and backward-incompatible changes, including intervening releases. Trace affected APIs, PHP signatures, DI/XML configuration, GraphQL contracts, template overrides, and JavaScript customizations into project code; concentrate on actual usage.
- Inventory local, Composer, quality, and security patches. Check whether target fixes supersede them or whether they still apply. Assess core/vendor modifications separately; do not automatically carry patches forward or remove them.
- Use Adobe's Upgrade Compatibility Tool when available for the edition and target; it is an Adobe Commerce tool, not a Magento Open Source prerequisite. Treat its findings as evidence to investigate, and state the limits of static analysis.

## Deliver the assessment

Give a concise readiness conclusion supported by a component table: current state, target requirement, evidence, and required action. Separate confirmed blockers from risks and unknowns. Link release/vendor sources and relevant project files; identify the smallest checks needed to resolve uncertainty. A clean static assessment alone cannot establish production readiness.

Provide an ordered plan covering dependency/service changes, code and patch remediation, and staging rehearsal. Tailor regression checks to affected checkout/payment, catalog/search, customer/Admin, integrations, Cron/consumers, and store scopes. Include data/schema migration effects, deployment constraints, and recovery requirements; distinguish code rollback from database restoration and account for writes after cutover.

Scale detail to the upgrade and provide effort ranges only when requested or useful, with assumptions. Assessment alone does not authorize package updates, `setup:upgrade`, reindexing, or deployment.

Consult Adobe's [upgrade scope guidance](https://experienceleague.adobe.com/en/docs/commerce-operations/upgrade-guide/prepare/scope), [system requirements](https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/system-requirements), and [Upgrade Compatibility Tool documentation](https://experienceleague.adobe.com/en/docs/commerce-operations/upgrade-guide/upgrade-compatibility-tool/overview) for the selected releases.
