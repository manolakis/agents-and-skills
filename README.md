# Agents and Skills

This repository defines a lightweight agent system for coordinating specialised reviews and guidance in a VS Code workflow. It pairs named agents with explicit skills, and documents how requests route to the correct reviewer, what each reviewer must check, and how automated sequences run for common lifecycle commands (for example, commit or ready for pr).

## Purpose

- Provide a consistent, repeatable review workflow across code quality, architecture, security, testing and commits.
- Encode review expectations as skills so agents follow the same rules and reporting format.
- Make automation predictable and auditable with clear routing and escalation rules.

## Repository Structure

- .github/agents/ contains agent definitions and their primary skills.
- .github/skills/ contains skill specifications, each with critical rules and workflows.
- AGENTS.md is the full coordination and routing guide.

## How It Works

1. A user issues a request such as @code-reviewer or @security.
2. The agent loads its required skills from .github/skills/.
3. The agent reviews the target changes and reports findings in a standard format.
4. If needed, the agent recommends escalation to another specialist.

## Automation Triggers

Certain commands run a full sequence automatically. These are documented in AGENTS.md and include:

- commit
- feature complete
- ready for pr
- fix complete
- hotfix

Each sequence runs the relevant agents in order and blocks on critical issues.

## Getting Started

- Read the full guide in AGENTS.md.
- Use @agent-name to request a focused review.
- Follow the reported findings and remediation advice.

## Contributing

- Keep documentation in British English.
- Update skills when rules or workflows change.
- Add new agents only when a distinct review domain exists.

