# Agent Blueprints

Reusable skills for Magento Open Source and Adobe Commerce work. Each skill captures decisions and pitfalls specific to a task, while leaving implementation size and tooling to the project.

## Skills

| Skill | Use for |
| --- | --- |
| [magento-backend-development](skills/magento-backend-development/SKILL.md) | Modules, PHP services, APIs, plugins, observers, and jobs |
| [magento-frontend-development](skills/magento-frontend-development/SKILL.md) | Storefront layouts, templates, styling, and interactions |
| [magento-code-review](skills/magento-code-review/SKILL.md) | Actionable findings in code changes |
| [magento-operations](skills/magento-operations/SKILL.md) | Runtime diagnosis, caching, queues, Cron, and deployments |
| [magento-database](skills/magento-database/SKILL.md) | Query plans, locks, schema changes, and data repairs |
| [magento-search](skills/magento-search/SKILL.md) | Catalog search, indexing, relevance, and search latency |
| [magento-testing](skills/magento-testing/SKILL.md) | Test strategy, unit/integration/API coverage, and backend test failures |
| [magento-browser-testing](skills/magento-browser-testing/SKILL.md) | Playwright customer journeys, local scenario data, and optional reusable browser tests |
| [magento-solution-design](skills/magento-solution-design/SKILL.md) | Feasibility, technical design, tradeoffs, and estimates |
| [magento-upgrade-assessment](skills/magento-upgrade-assessment/SKILL.md) | Target-release compatibility, upgrade blockers, and validation planning |
| [magento-security-audit](skills/magento-security-audit/SKILL.md) | Security assessments, hardening, and incident investigation |

## Use with Codex

Each `skills/<name>/` folder is a self-contained skill with a `SKILL.md` entry point. Copy selected folders into `.agents/skills/` in a Magento project, or `~/.agents/skills/` for use across projects. For example, from the target project:

```bash
mkdir -p .agents/skills
cp -R /path/to/agent-blueprints/skills/magento-backend-development .agents/skills/
```

Replace `/path/to/agent-blueprints` with your checkout path. A symlink to an individual skill folder also works for local development; commit copied folders when teammates need a portable setup.

Codex can select installed skills by their descriptions, or you can invoke one explicitly:

```text
Use $magento-backend-development to fix duplicate processing in this import job.
Use $magento-code-review to review the checkout changes on this branch.
Use $magento-browser-testing to test this feature as a customer with local sample data and save reusable developer-only tests in the module.
```

Select skills for the work at hand; they are not a required sequence of roles. Ordinary implementation can include testing without requiring a separate QA workflow. See the [official skill documentation](https://learn.chatgpt.com/docs/build-skills) for discovery and installation details.

The earlier flat Markdown role prompts have been replaced by these skill folders and task names. Update any saved prompts or integrations referencing the old names or paths.

## Project conventions

Keep durable project-specific instructions in the Magento project's `AGENTS.md`: module namespace and ownership, supported versions, local/container command wrappers, style rules, build/test commands, and boundaries for environment access. Record deliberate defaults there too, such as required feature toggles or manual-only database writes.

Each skill can be copied on its own and follows project instructions when present. The skills do not assume a particular theme, infrastructure topology, logging setup, or test framework version. Documentation, abstractions, CLI commands, and operational tooling should follow the change's needs and the project's established conventions.

## Warden project template

Use [templates/magento-warden/AGENTS.md](templates/magento-warden/AGENTS.md) as a starting point for Magento projects running in Warden. Copy it to the target project's root as `AGENTS.md`, or merge it into existing instructions. Check the project's mounts, routing, service overrides, and test commands when adapting it.

The template covers container-only PHP execution, database/search access, storefront checks, browser testing, and logs. It contains no customer names, custom module identifiers, catalog examples, or machine-specific paths. Environment access belongs in project instructions because it applies across tasks; install the skills separately rather than maintaining a static skill catalog in `AGENTS.md`.

See [Warden usage](https://docs.warden.dev/usage.html), [environment customization](https://docs.warden.dev/environments/customizing.html), and [Codex project instructions](https://learn.chatgpt.com/docs/agent-configuration/agents-md) for the underlying configuration and discovery behavior.

## Maintaining the library

- Add instructions that prevent a concrete mistake or encode a deliberate workflow; omit generic expertise claims and repeated checklists.
- Keep descriptions precise enough to select the right task. Add supporting references or scripts only when they have a concrete use.
- Verify technical guidance against installed source and official release-specific documentation. Do not freeze a universal dependency version into a skill.
- Validate skill frontmatter and links after edits. Evaluate meaningful behavior with realistic requests, such as a small bug fix, a new integration, a storefront change, and a code review. Check that each gets an appropriate implementation and verification scope.

## License

MIT — see [LICENSE](LICENSE).
