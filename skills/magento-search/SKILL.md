---
name: magento-search
description: Diagnose or tune Magento catalog search, indexing, and relevance. Use for missing products, incorrect ranking, search latency, or search-adapter changes involving the installed search provider.
---

# Magento search

## Identify the search path

Establish Magento edition/version, store scope, active provider, installed adapter/extensions, and engine version. Distinguish native catalog search from Live Search or third-party implementations. Use the provider's actual index aliases, mappings, and document identifiers rather than guessing names or assuming SKU is the document ID.

Capture a failing query, expected products/order, actual results, and applicable visibility, stock, category, website, and customer context.

## Trace the failure

- For missing products, follow source data and eligibility through indexer state, Cron/consumers, the indexed document, the outgoing request, and storefront filtering. Check connectivity and cluster errors where an engine is involved, without changing cluster settings.
- For ranking, inspect the generated query, analyzers, attribute weights, and provider rules. Use search responses or explain/profile APIs when available to test a specific hypothesis. Hard constraints usually belong in filter context; choose scoring clauses deliberately.
- For latency, separate engine time from PHP/database hydration and indexing contention. Inspect mappings and aggregation behavior before tuning memory or concurrency. Do not bypass Magento pricing, visibility, or inventory logic by hydrating storefront products solely from engine `_source`.

## Implement and verify

Choose the smallest supported configuration or adapter extension for the installed provider. Confirm extension classes and XML schemas in its source/documentation before proposing mapping files. Synonyms, fuzziness, n-grams, and fallback matching need representative relevance tests; zero results can be the correct outcome.

Check release/indexer support and resource capacity before changing parallelism, batch sizes, or indexer mode. Mapping/analyzer changes may require a replacement index and reindex; plan the operational impact before a live rebuild.

Compare representative queries before and after, including exact SKU/name, ambiguous terms, filters, and relevant store scopes. Report measured behavior and the configuration or commands needed to reproduce it, with credentials redacted.

Consult Adobe's [system requirements](https://experienceleague.adobe.com/en/docs/commerce-operations/installation-guide/system-requirements) and [indexer optimization guidance](https://developer.adobe.com/commerce/php/development/components/indexing/optimization) when compatibility or indexing is involved.
