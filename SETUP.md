# Setup

This repository is the public Studio skill bundle. It contains the core Studio workflow plus a small set of generic companion skills.

## Install The Root Skill

Copy `SKILL.md` and `references/` into your target skill directory, or clone the repository and point your agent runtime at it.

```bash
git clone https://github.com/xunova739/studio-skill-adapters.git
```

## Codex Project Install

```bash
mkdir -p /path/to/project/.codex/skills
cp -R .codex/skills/autonomous-studio-codex /path/to/project/.codex/skills/
```

Then reload Codex if the skill list is cached.

## Claude Code Project Install

```bash
mkdir -p /path/to/project/.claude/skills
cp -R .claude/skills/autonomous-studio-claude /path/to/project/.claude/skills/
```

Hooks are not installed by default. Review any hook before enabling it in a project.

## Optional Companion Skills

The `skills/` directory includes generic workflow helpers:

- `demand-discovery`
- `idea-exploration`
- `pm-spec`
- `serial-agent-handoff`
- `agents-map`
- `agent-context-authoring`
- `agents-md-slim`
- `excalidraw-diagram-skill`
- `luban`

Internal platform adapters are intentionally excluded from this public bundle.
