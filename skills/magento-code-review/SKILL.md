---
name: magento-code-review
description: Review Magento code changes for correctness, security, performance, and compatibility. Use for review requests; focus on actionable findings in the changed code and affected behavior.
---

# Magento code review

## Review scope

Read the diff, surrounding code, callers, and relevant tests. Establish the supported versions and project conventions. Trace a suspected defect to a concrete execution path before reporting it; distinguish changed-code regressions from existing issues.

## Magento checks

- Check plugin signatures against the intercepted method and plugin type. Verify continuation/return behavior in `around` plugins, observer area coverage, DI wiring, and dependencies on installed modules.
- Trace authorization and data ownership through controllers, APIs, GraphQL resolvers, and UI data providers. Admin controllers can use `ADMIN_RESOURCE`; a custom `_isAllowed()` override is not universally required. Check request method and CSRF handling where applicable.
- Check escaping at output sinks and parameter binding at SQL boundaries. SQL itself is not evidence of injection, and ViewModel data does not need HTML escaping until rendered.
- Look for repeated entity loads, unbounded collections, and missing pagination with a credible data-size impact. Check cache keys, identities, invalidation, and customer/store context at the actual cache integration point.
- For jobs and integrations, check retries, duplicate execution, partial failures, transaction boundaries, and restoration of store emulation. Verify schema/data changes against existing installations.
- Judge direct `ObjectManager` use, missing abstractions, and style choices by their actual impact. Do not equate a convention violation with an exploitable vulnerability.

## Findings

Order findings by impact. For each, give a short title, file and line, the triggering scenario, and the resulting failure. Include a focused fix suggestion when useful; provide a full rewrite only when requested. Label uncertain conclusions with the missing evidence.

If no actionable defects are found, say so and identify material verification gaps. Keep optional style suggestions separate and omit arbitrary readiness scores. Review is read-only unless fixes are requested; run relevant checks only when practical and safe for the target environment.

Consult Adobe's [authorization model](https://developer.adobe.com/commerce/php/development/security/authorization) when assessing access control.
