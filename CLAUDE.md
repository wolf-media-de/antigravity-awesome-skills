# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**antigravity-awesome-skills** is a curated collection of 1,400+ agentic skills for Claude Code, Gemini CLI, Cursor, Antigravity, and other AI coding tools, plus an installer CLI (`antigravity-awesome-skills` binary).

## Commands

```bash
# Install dependencies
npm install

# Validate all skills (frontmatter, schema, name-directory match)
npm run validate

# Validate with stricter quality rules (legacy skills may not pass)
npm run validate:strict

# Run test suite (infrastructure, references, docs security)
npm test

# Docs security scan — required for skills with shell commands, network access, or credentials
npm run security:docs

# Full sync chain: validate → plugin-compat:sync → index → bundles:sync → sync:metadata
npm run chain

# Full build (chain + catalog regeneration)
npm run build

# PR preflight check (maintainer use; runs chain + catalog + audits)
npm run pr:preflight

# Web app
npm run app:setup    # one-time setup
npm run app:dev      # dev server
npm run app:build    # production build
npm run app:test     # component tests
```

Python tools can also be run directly: `python3 tools/scripts/validate_skills.py`

## Architecture

### Skill format

Each skill lives at `skills/<name>/SKILL.md`. Required YAML frontmatter:

```yaml
---
name: skill-name          # must exactly match the directory name
description: "One sentence describing what this skill does"
risk: safe                # safe | low | medium | high | offensive | critical | unknown
source: community
date_added: "YYYY-MM-DD"
---
```

Required sections: `## When to Use This Skill`, `## How It Works`, `## Examples`. Soft cap ~500 lines. The canonical template is at `docs/contributors/skill-template.md`.

### Generated artifacts — never include in contributor PRs

These files are regenerated on `main` after merge and must not appear in PR diffs:

- `CATALOG.md`
- `skills_index.json`
- `data/skills_index.json`, `data/catalog.json`, `data/bundles.json`, `data/aliases.json`

### Key directories

- **`skills/`** — Single source of truth for all skill content.
- **`tools/scripts/`** — Python validation and maintenance scripts, each invocable via `npm run <name>` through the `run-python.js` wrapper. Key scripts: `validate_skills.py`, `audit_skills.py`, `generate_index.py`, `plugin_compatibility.py`, `build-catalog.js`.
- **`tools/bin/`** — CLI installer (`install.js`), published as the `antigravity-awesome-skills` binary.
- **`tools/lib/`** — Shared Node.js library used by the CLI.
- **`apps/web-app/`** — Vite-based frontend for browsing and searching skills. Reads from `skills_index.json`. Requires `npm run app:setup` before first use.
- **`data/`** — Generated JSON bundles consumed by the web app and external integrations. Do not edit manually.
- **`plugins/`** — Plugin-compatibility output directory; synced from `skills/` by `npm run plugin-compat:sync`.
- **`skill_categorization/`** — Category taxonomy data used by the index generator.
- **`docs/contributors/`** — Contributor templates and guides, including the canonical `skill-template.md`.
- **`.agents/`, `.claude-plugin/`** — Harness-specific integration files.

### Validation pipeline

`npm run chain` runs these steps in order:
1. `validate` — frontmatter schema + name/directory consistency
2. `plugin-compat:sync` — synchronise plugin-compatibility layer
3. `index` — regenerate `skills_index.json`
4. `bundles:sync` — regenerate editorial bundles
5. `sync:metadata` — update repository metadata

Risk labels are reconciled separately via `npm run sync:risk-labels` (maintainer operation).

## Conventions

- Skill directory name must exactly match the frontmatter `name` field.
- Commit format: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`.
- PRs must be source-only (no generated artifacts). Enable **Allow edits from maintainers** on every PR.
- Skills with shell commands, network access, credentials, or destructive guidance require `npm run security:docs` and `<!-- security-allowlist: ... -->` comments on any intentional high-risk patterns.
- `risk: unknown` is acceptable for genuinely unclassified legacy content; maintainers reconcile after merge.
