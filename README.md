# coach-skills

Shared custom Claude Code skills made by the **Coach** team.

Each folder is a self-contained skill (a `SKILL.md` plus references). Skills are
invoked via their slash command or auto-triggered based on their `description`.

## Available skills

| Skill | Invocation | Purpose |
|-------|------------|---------|
| [`code-review`](./code-review/SKILL.md) | `/pr-review` | Automated PR review: detects intent, cross-checks Linear specs, dispatches expert agents, classifies findings by severity, and posts inline comments on the exact lines. Runs autonomously in GitHub Actions. |

## Skill layout

```
<skill>/
├── SKILL.md        # frontmatter (name + description) + instructions
└── references/     # detailed docs loaded on demand
```

The `description` frontmatter determines when a skill triggers — keep it specific
about the usage context.
