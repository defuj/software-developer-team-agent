# GitHub Pseudo-Skills

This directory emulates OpenCode-style skills for GitHub custom agents.

Because GitHub agents do not have native dynamic skill loading, each skill is represented as a markdown playbook that the agent should consult based on task context.

## Core Pseudo-Skills

- `coding-standards/SKILL.md`
- `frontend-patterns/SKILL.md`
- `frontend-design/SKILL.md`
- `web-design-guidelines/SKILL.md`

## Contextual Pseudo-Skills

- `nuxt-ui/SKILL.md`
- `vercel-composition-patterns/SKILL.md`
- `building-components/SKILL.md`
- `security-review/SKILL.md`
- `tdd-workflow/SKILL.md`

## Usage Pattern

1. Start with core pseudo-skills.
2. Add contextual pseudo-skills only when relevant.
3. Prefer repository-local files in `.github/skills/` over external references.
4. Keep this folder portable so `.github/agents/*` works without `.opencode/`.

## Maintenance

- If source skills evolve, sync updates into this folder.
- Avoid deleting referenced skill files without updating agent mappings.
