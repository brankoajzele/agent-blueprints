---
name: magento-browser-testing
description: Exercise Magento customer journeys in a real browser with Playwright, including local scenario data and failure evidence. Use for on-demand storefront verification or requested reusable developer-only browser tests.
---

# Magento browser testing

## Establish the scenario and mode

Read the feature, acceptance criteria, and existing browser tooling. Resolve the development/test base URL, storefront, store scope, and customer role from project configuration. Use the available Playwright tools or installed runner; identify missing prerequisites before claiming execution.

Support two modes: on-demand verification with temporary automation, or reusable tests when requested. Reuse the project's harness. For module-local tests without an established location, `Test/Browser/` is a reasonable convention; it is not a Magento-required path. A browser check alone does not require installing a permanent test framework into the project.

## Prepare realistic local data

- Identify the minimal customers, products, stock, configuration, and other state needed for the scenario. Seed through supported application services, APIs, or development CLI helpers. Keep setup out of module installation, data patches, public routes, and production execution.
- Integration fixtures target their isolated test database; browser fixtures prepare the running development/test store. Share data builders when useful, but keep lifecycles separate. Never run the integration-test installer or database reset against the browser target. Browser-context isolation does not isolate Magento database state.
- Namespace created records per run or worker, track ownership, and clean up only those records. Restore changed configuration at its original scope. Make setup/cleanup repeatable, account for partial failure, and wait for required indexing or background processing before browser assertions.
- Use test accounts and sandbox/offline payment and communication services for scenarios with external effects. Keep credentials and authenticated browser state outside version control. Isolate mutable carts/accounts across parallel tests, or run affected scenarios serially.

## Exercise the customer journey

Perform the feature's customer-facing actions through the UI. APIs may prepare prerequisites or verify persisted outcomes, but should not bypass the interaction under test. Keep mocked dependencies explicit so the result does not imply an untested integration passed.

Use role/label locators or stable test IDs and Playwright's waiting/assertion mechanisms rather than fixed sleeps or forced clicks that conceal broken interactions. Assert the actual outcome, including relevant rejection and recovery paths. For personalized or cached features, test the applicable guest/customer, store, and warm-cache states. Use a fresh login when authentication itself is under test.

Capture useful screenshots/traces on failure and inspect related console/network errors. Handle artifacts as potentially sensitive. Reproduce and diagnose failures before adding retries or weakening assertions.

## Save and report

For reusable tests, keep runner dependencies and seed helpers in developer tooling. Keep tests/builders portable and suitable for version control; ignore local configuration, credentials, saved sessions, and generated artifacts. Document the focused run command, prerequisites, and setup/cleanup behavior in the existing module/project documentation.

Report the target environment, scenarios and assertions actually exercised, pass/fail/blocked results, useful evidence, and any remaining fixture data or configuration changes. Missing prerequisites or incomplete cleanup must remain visible; do not count an unexecuted scenario as passing.

Consult Playwright's [best practices](https://playwright.dev/docs/best-practices), [fixtures](https://playwright.dev/docs/test-fixtures), and [authentication guidance](https://playwright.dev/docs/auth) when implementing the runner or setup lifecycle.
