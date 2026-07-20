# AI Code Migration Skill

A Hermes Agent skill distilled from Anthropic's article:

https://claude.com/blog/ai-code-migration

The central idea: **fix the migration loop, not individual generated-code symptoms.** The skill turns the article's six-step process into a reusable workflow for AI-assisted language ports, framework migrations, parity-driven rewrites, and large refactors.

## Contents

- `code-migration-skill/SKILL.md` — installable Hermes skill.
- `SUMMARY.md` — concise extraction of the method.
- `LICENSE` — MIT.

## Install

Copy the skill directory into your Hermes profile:

```bash
mkdir -p ~/.hermes/skills/software-development
cp -R code-migration-skill ~/.hermes/skills/software-development/ai-code-migration
```

## Attribution

Created for and signed by GitHub user id: `Lucien`.
Source article © Anthropic; this repository is a distilled procedural skill, not a copy of the original article.