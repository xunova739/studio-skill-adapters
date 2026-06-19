# Autonomous Studio Skill

A project-local skill for running a structured product-to-development workflow across Codex and Claude Code.

This repository contains the actual skill, not just documentation:

- `SKILL.md` is the canonical Studio skill.
- `.codex/skills/autonomous-studio-codex/` is the Codex install form.
- `.claude/skills/autonomous-studio-claude/` is the Claude Code install form.
- `skills/` contains generic companion skills used by the Studio workflow.

No repository URL is required inside the skill frontmatter. The GitHub repository itself is the distribution source.

## What It Does

Autonomous Studio keeps a project moving through a visible workflow:

```text
demand discovery
  -> PRD.html draft
  -> human confirmation
  -> PRD.json structuring
  -> technical planning
  -> agent-skill development
  -> verification
  -> review and fix
  -> deploy or handoff
  -> feedback loop
```

The important boundary is:

```text
PRD.html first
  -> user confirms
  -> then PRD.json
```

`PRD.html` is a separately triggered review artifact. The skill should not generate `PRD.json` at the same time. `PRD.json` is created only after the user confirms the HTML PRD.

## Install

### As a generic skill

Copy the whole repository or the root `SKILL.md` plus `references/` into your skill directory.

```bash
git clone https://github.com/xunova739/studio-skill-adapters.git
```

### Codex

```bash
mkdir -p /path/to/project/.codex/skills
cp -R .codex/skills/autonomous-studio-codex /path/to/project/.codex/skills/
```

### Claude Code

```bash
mkdir -p /path/to/project/.claude/skills
cp -R .claude/skills/autonomous-studio-claude /path/to/project/.claude/skills/
```

Reload the target agent environment if its skill list is cached.

## Trigger Examples

- `studio, what stage is this project in?`
- `Generate PRD.html only. Do not write PRD.json yet.`
- `The PRD.html is approved. Now write PRD.json.`
- `Move from PRD to agent development.`
- `Verify this feature before review.`
- `Use the Studio workflow from demand to launch.`

Chinese prompts are also supported:

- `studio，帮我判断这个项目现在处于哪个阶段`
- `单独生成 PRD.html，先不要写 PRD.json`
- `PRD.html 已确认，现在写 PRD.json`
- `从需求到上线，按 Studio 流程推进`

## Files

| Path | Purpose |
|---|---|
| `SKILL.md` | Canonical Studio skill. Use this as the main public skill entry. |
| `references/studio-pipeline.md` | Detailed stage rules loaded only when needed. |
| `.codex/skills/autonomous-studio-codex/` | Codex-specific install form using `AGENTS.md`. |
| `.claude/skills/autonomous-studio-claude/` | Claude Code-specific install form using `CLAUDE.md`; hooks are optional and reviewed. |
| `skills/demand-discovery/` | Requirement discovery helper. |
| `skills/idea-exploration/` | Early-stage idea and feasibility exploration. |
| `skills/pm-spec/` | PRD authoring helper. |
| `skills/serial-agent-handoff/` | Agent development handoff helper. |
| `skills/agents-map/` | Multi-project agent context mapping helper. |
| `skills/agent-context-authoring/` | AGENTS.md / CLAUDE.md authoring helper. |
| `skills/agents-md-slim/` | AGENTS.md slimming helper. |
| `skills/excalidraw-diagram-skill/` | Editable Excalidraw diagram helper. |
| `skills/luban/` | Skill refinement helper. |
| `docs/workflow.md` | Human-readable workflow explanation. |
| `examples/prompts.md` | Example trigger prompts. |
| `examples/status.example.json` | Example status file. |
| `evals/evals.json` | Root skill eval prompts. |
| `SECURITY.md` | Public safety boundary. |

## Public Bundle Boundary

This public bundle intentionally excludes internal platform adapters, deployment tools, company memory stores, private registries, and organization-specific workflow scripts. Those are not required for the Studio core workflow.

The Studio core depends on the workflow contract, not on a specific company platform:

```text
demand discovery
  -> PRD.html review
  -> human approval
  -> PRD.json
  -> implementation plan
  -> agent development
  -> verification
```

## Safety Boundary

The skill does not:

- install hooks automatically;
- embed credentials in skill metadata;
- copy sessions, decision logs, checkpoints, or memory dumps;
- push code, deploy, or call external services without the active project policy or explicit user instruction;
- make black-box decisions for recruiting, screening, ranking, or other high-impact workflows.

For multi-business-line systems, prefer reusable configuration over one prompt per job:

```text
global rubric
  + business-line policy/config
  + job-specific requirements
  + evidence extraction
  + scoring reason
  + human review and override
```

## Validation

Before publishing:

- JSON eval files were parsed with `jq`.
- The repository was scanned for local machine paths, internal domains, credential-shaped URLs, and common key patterns.
- `PRD.html -> confirmation -> PRD.json` is documented in both `SKILL.md` and `docs/workflow.md`.
