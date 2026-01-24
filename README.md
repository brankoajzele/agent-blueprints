# Agent Blueprints

High-performance AI agent skill definitions and system prompts for the Codex CLI. Optimized for senior-level engineering workflows, these blueprints enforce strict architectural standards, performance guardrails (N+1 prevention), and robust observability (logging/CLI parity) for complex platforms like Magento 2. Professional-grade AI orchestration.

## What this repo is
- A curated library of agent skills and system prompts
- Opinionated guidance for senior-level engineering workflows
- A place to codify repeatable, high-signal agent behavior

## Structure
- `skills/` — Skill definitions and supporting references

## Usage
1. Add or update skill folders under `skills/`.
2. Keep each skill self-contained with a `SKILL.md` entry point.
3. Prefer concise, deterministic instructions that are easy to apply.

## Conventions
- Keep instructions actionable and testable.
- Avoid duplicating guidance across skills; link when needed.
- Favor explicit constraints over broad suggestions.

## License
MIT — see `LICENSE`.
