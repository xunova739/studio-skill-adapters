# Source Surface Audit

This repository is a public-safe Studio bundle, not a raw dump of the original local source tree.

The original source tree contained both Studio core material and private runtime adapters. Public publishing keeps the workflow core and excludes internal platform bindings, local automation state, and destructive default automation.

## Included Core

These surfaces are part of the public Studio core:

| Source Surface | Public Location | Reason |
|---|---|---|
| Studio router | `SKILL.md` | Canonical workflow router. |
| Codex adapter | `.codex/skills/autonomous-studio-codex/` | Codex install form. |
| Claude Code adapter | `.claude/skills/autonomous-studio-claude/` | Claude Code install form. |
| Decision Agent prompt | `decision-agent-prompt.md` | Isolated state-file-driven route judgment. |
| Stage workflow details | `docs/workflow.md`, `references/studio-pipeline.md` | PRD, implementation, verification, archive gates. |
| Companion skills | `skills/demand-discovery`, `skills/idea-exploration`, `skills/pm-spec`, `skills/serial-agent-handoff`, `skills/agents-map`, `skills/agent-context-authoring`, `skills/agents-md-slim`, `skills/excalidraw-diagram-skill`, `skills/luban` | Generic helpers used by Studio stages. |

## Excluded Source Surfaces

The following original source surfaces are intentionally not shipped in this public bundle:

| Source Surface | Reason |
|---|---|
| Root `AGENTS.md` from the original source | It indexed internal platform skills and organization-specific adapters. The public README replaces it. |
| Original `ARCHITECTURE.md` and `autonomous-engine/` runtime files | They described a local Claude/WSL/watchdog/phone-notification deployment rather than portable Studio behavior. The public bundle keeps the Decision Agent prompt and Studio workflow rules instead. |
| `.claude/hooks/`, `hooks/install-studio-hooks.sh`, and hook installer scripts | They modify user hook settings and are not safe as default public install behavior. Public hooks must be opt-in, reviewed, and narrow. |
| `config/settings.json.example`, scheduled tasks, phone-notify config, watchdog scripts, and Termux listener | They are local runtime examples with machine-specific assumptions. |
| Old `studio-pipeline.md` source file | It contains legacy auto-push/deploy assumptions and internal tool references. The public `docs/workflow.md` and `references/studio-pipeline.md` supersede it. |
| `scripts/action-dispatch.md` from source | It includes old default `git push` behavior. Public Studio treats push/PR/deploy as explicit publishing actions. |
| Internal platform skills and related deployment scripts | These bind to company or internal environments and are not required for the portable Studio core. |
| Local memory stores, checkpoints, sessions, decision logs, and calibration data | Runtime state is not publishable source. |

## Safety Rule

When adding material from the original source tree, first classify it:

1. `core`: portable Studio workflow, prompt, eval, or generic companion skill.
2. `adapter`: runtime-specific but potentially publishable after sanitization.
3. `internal`: private platform, company workflow, local state, credential, or environment binding.

Only `core` files should be published by default. `adapter` files need public-safe rewriting. `internal` files stay out of the public repository.
