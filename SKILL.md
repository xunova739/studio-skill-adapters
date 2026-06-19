---
name: autonomous-studio
description: Studio workflow router for demand discovery, PRD.html review, PRD.json structuring, agent development, verification, review, and deployment handoff. Use this skill whenever the user says studio, autonomous mode, keep working, next step, full workflow, from demand to launch, write PRD, generate PRD.html, write PRD.json after approval, start development, verify, review, or deploy. This skill supports both Codex and Claude Code project-local runtime files.
---

# Autonomous Studio

Autonomous Studio is a state-aware workflow router. It does not replace the product owner or developer. It keeps the project moving through explicit stages and makes the next safe action clear.

Core rule:

```text
AI assists. Humans decide.
```

## Runtime Selection

When the skill is triggered, detect the current runtime and use the matching local context file:

| Runtime | Context File | Skill Form |
|---|---|---|
| Codex / Codex Desktop | `AGENTS.md` | `.codex/skills/autonomous-studio-codex/SKILL.md` |
| Claude Code | `CLAUDE.md` | `.claude/skills/autonomous-studio-claude/SKILL.md` |

If the runtime is unclear, use this root skill as the canonical behavior and avoid installing hooks.

## Stage Order

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

Important boundary:

- `PRD.html` is a separately triggered human review artifact.
- `PRD.json` is an implementation-facing structured contract.
- Do not create `PRD.json` in the same step as `PRD.html`.
- Create `PRD.json` only after the user confirms the `PRD.html`.

## Activation Steps

1. Read the project status file:
   - prefer `planning/status.json`;
   - fallback to `.planning/status.json`;
   - if neither exists, infer the current stage from artifacts and explain the inference.
2. Report:
   - current stage;
   - evidence;
   - completed artifacts;
   - smallest useful next action;
   - steps that can be skipped for small tasks.
3. If the user asks to enable Studio locally, write a short marker-bounded block:
   - Codex: write to `AGENTS.md`;
   - Claude Code: write to `CLAUDE.md`;
   - do not install hooks unless explicitly requested and reviewed.
4. Continue only within the current confirmed stage. Do not silently jump from PRD.html to PRD.json or from PRD.json to coding.

## Stage Detection

Check from latest to earliest:

1. deploy record or release handoff exists -> `deploy_or_done`
2. review notes or review fix commits exist -> `review`
3. verification output exists -> `verification`
4. code changes exist for this task -> `development`
5. technical plan exists -> `technical_planning`
6. `planning/prd.json` or `.planning/prd.json` exists -> `prd_json_ready`
7. confirmed `planning/prd.html` or `.planning/prd.html` exists -> `prd_confirmation`
8. draft `planning/prd.html` or `.planning/prd.html` exists -> `prd_html_review`
9. requirements notes exist -> `prd_html_draft`
10. none -> `demand_discovery`

## Stage Routing

| Stage | Default Action | Main Artifact |
|---|---|---|
| `demand_discovery` | clarify business goal, users, scope, constraints, and non-goals | `requirements.md` or discovery brief |
| `prd_html_draft` | create the readable PRD page | `PRD.html` |
| `prd_html_review` | wait for user comments or approval | confirmed `PRD.html` |
| `prd_confirmation` | apply requested edits until approved | confirmed `PRD.html` |
| `prd_json_ready` | produce implementation-facing structure only after approval | `PRD.json` |
| `technical_planning` | split modules, data flow, permissions, tests, rollback | technical plan |
| `development` | invoke the relevant agent skill and implement narrowly | code diff |
| `verification` | run tests, lint, typecheck, Playwright, or manual checks | verification notes |
| `review` | prioritize bugs, regressions, security/data risks, missing tests | review notes and fixes |
| `deploy_or_done` | deploy only when authorized, otherwise produce handoff | release or handoff doc |

## Local Context Block

When enabling Studio, keep the injected block short.

```markdown
<!-- STUDIO:BEGIN -->
## Studio Workflow

- State: prefer `planning/status.json`; fallback `.planning/status.json`.
- Flow: demand discovery -> PRD.html -> human confirmation -> PRD.json -> technical plan -> development -> verification -> review -> deploy/handoff.
- `PRD.html` is a separate review artifact. Do not write `PRD.json` until the user confirms it.
- AI assists; humans decide. High-impact recommendations need evidence, score/reason, and override path.
- Do not install hooks, push, deploy, or call external services unless the current project policy or user explicitly allows it.
<!-- STUDIO:END -->
```

## Multi-Business-Line Prompting

For recruiting, screening, ranking, routing, or other multi-business-line workflows, do not tune one prompt per job by hand. Prefer layered configuration:

```text
global rubric
  + business-line policy/config
  + job-specific requirements
  + evidence extraction
  + scoring reason
  + human review and override
```

The skill should help create reusable prompt/config surfaces, not black-box decisions.

## Detailed Reference

Read `references/studio-pipeline.md` when executing a concrete stage or when the user asks how the whole flow should work.
