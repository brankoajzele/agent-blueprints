---
name: magento-solution-design
description: Turn Magento business requirements into a feasible technical approach, tradeoffs, and implementation scope. Use for planning, feature feasibility, integration design, or effort estimates before implementation.
---

# Magento solution design

## Establish the requirement

Identify the business outcome, acceptance criteria, and constraints that affect the decision. Infer current edition/version, installed modules, storefront, integrations, and deployment from available project evidence. Ask only for missing facts that materially change the recommendation.

## Compare viable approaches

- Check existing native and installed functionality before custom development. Verify edition, licensing, version, and hosting availability rather than assuming an Adobe Commerce feature exists in Magento Open Source.
- Compare configuration, a maintained extension, and custom implementation when they are credible alternatives. Judge extensions by compatibility, support, data handling, maintenance, and fit; vendor reputation alone is insufficient.
- Trace data ownership, store/website scope, authorization, API contracts, synchronization, and failure recovery for integrations. Choose synchronous or asynchronous processing from consistency, latency, volume, and operational constraints.
- Assess effects on checkout, pricing, inventory, caching, and indexing where relevant. Identify migration and upgrade constraints. Size infrastructure from expected workload rather than assuming a cluster is needed.
- Identify applicable privacy, payment, and accessibility obligations when the requirement touches them. Do not claim compliance from a design sketch.

## Deliver a proportionate design

For a small request, give a concise recommendation with rationale and affected components. For a larger change, include acceptance criteria, meaningful alternatives, interfaces/data flow, implementation stages, and material risks or unanswered questions.

Provide estimates only when requested or useful for planning, as ranges tied to stated assumptions, dependencies, and uncertainty. Distinguish engineering effort from elapsed delivery time. Avoid numerical native/custom thresholds, arbitrary buffers, and unsupported precision.

Make the handover concrete enough to implement: affected modules, important extension points, and the behavior that needs validation. Planning alone does not authorize implementation or external changes.
