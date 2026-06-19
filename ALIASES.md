# Skill Aliases

Use these aliases in durable docs instead of repeating concrete paths.

| Alias | Skill | Runtime |
|---|---|---|
| `@studio-codex` | `.codex/skills/autonomous-studio-codex` | Codex / Codex Desktop |
| `@studio-claude` | `.claude/skills/autonomous-studio-claude` | Claude Code |

Rules:

- Keep aliases stable even if the directory name changes later.
- Do not use local machine paths in public documentation.
- Do not encode GitHub credentials in alias metadata.
