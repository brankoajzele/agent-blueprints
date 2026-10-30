---
name: magento-testing
description: Design, add, or debug Magento unit, integration, and API tests and choose regression coverage. Use for test strategy or backend test failures; browser-driven customer journeys belong to browser testing.
---

# Magento testing

## Choose the test boundary

Read the changed behavior, existing tests, Composer lockfile, and test configuration. Use the installed PHPUnit and project runner; do not assume a framework version from the Magento release family alone. For Playwright customer journeys and local browser scenario data, use `magento-browser-testing` when available; backend tests remain independently runnable.

Test the behavior most likely to regress:

- Use unit tests for calculations, decisions, mappings, and error handling that can be isolated from Magento runtime.
- Use integration tests for persistence, declarative schema/data patches, DI configuration, plugins/events, and store-scoped behavior that mocks cannot establish.
- Use API tests for authorization and request contracts, and browser tests when customer interactions, rendering, or cache/private-content behavior are the boundary that matters.

Choose coverage by risk. An `Api/` implementation does not automatically need a mock-heavy unit test; a thin adapter may be better covered through integration.

## Write and run

For a regression, reproduce the failure first when practical, then verify the corrected behavior. Follow existing fixtures and isolation conventions. Mock external boundaries in unit tests and assert observable outcomes; avoid testing private implementation details or reproducing the implementation in assertions.

Cover relevant failure and boundary cases, such as duplicate job delivery, partial integration failure, store scope, permissions, or rounding. Use data providers when they clarify a real input matrix. Refactor for testability only when it improves the design within scope.

Use a configured, isolated test database for integration tests. Do not point setup/cleanup routines at a development or production database. Keep remote dependencies controlled and restore state created by the test.

Run the narrowest relevant suite, followed by required project checks. Report the exact command, result, and any untested boundary. If infrastructure prevents execution, state that and provide the necessary setup or focused manual verification; do not claim the tests passed. Document new test setup in the existing location when needed.
