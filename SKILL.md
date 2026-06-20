---
name: autonomous-studio
description: Studio workflow router for turning rough product, workflow, or skill-development requests into staged, human-approved delivery artifacts before implementation. Use when the user asks for studio, autonomous workflow, demand-to-launch, PRD.html, PRD.json after approval, agent development, verification, review, handoff, skill optimization, or "what is the next step?" Do not use for simple Q&A, direct bug fixes, typo edits, or one-shot research unless the user explicitly asks to enter Studio flow. Supports both Codex and Claude Code project-local runtime files.
---

# Autonomous Studio

Autonomous Studio is a state-aware workflow router. It does not replace the product owner or developer. It keeps the project moving through explicit stages and makes the next safe action clear.

Core rule:

```text
AI assists. Humans decide.
```

Core principle: Studio turns unclear intent into traceable artifacts. AI may draft, inspect, implement, and verify; humans approve intent, business rules, publishing, irreversible actions, and high-impact decisions.

## Trigger Boundary

Use Studio when the user wants a staged workflow, not just an answer. Good trigger contexts include:

- rough product, system, workflow, or automation ideas that need demand discovery;
- PRD.html drafting and human confirmation before PRD.json;
- agent/skill/prompt/workflow development that should be repeatable and verifiable;
- multi-business-line systems where shared rules and local configuration must be separated;
- delivery flows that need planning, implementation, verification, review, handoff, or archive.

Do not enter Studio flow for:

- simple typo, copy, or formatting edits;
- direct bug fixes with clear expected behavior and no product/process ambiguity;
- small config changes without workflow or business impact;
- one-shot explanation, comparison, search, or code review;
- tasks where the user explicitly says to only discuss, only answer, or not write artifacts.

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
  -> implementation or skill/eval development
  -> verification
  -> review and fix
  -> deploy or handoff
  -> archive and feedback loop
```

Important boundary:

- `PRD.html` is a separately triggered human review artifact.
- `PRD.json` is an implementation-facing structured contract.
- Do not create `PRD.json` in the same step as `PRD.html`.
- Create `PRD.json` only after the user confirms the `PRD.html`.
- "Looks okay", "let me see", "continue discussing", or silence is not confirmation. Treat explicit approval, explicit revision acceptance, or an explicit request to generate structured PRD as confirmation.

## Change Artifacts and Archive

For non-trivial work, prefer one change folder per demand:

```text
planning/changes/<change-id>/
  prd.html
  prd.json
  design.md
  tasks.md
  verify.md
  retrospective.md
```

If the project already uses `.planning/changes/`, keep that convention. Completed work can move to:

```text
planning/changes/archive/YYYY-MM-DD-<change-id>/
```

This follows the OpenSpec-style idea that each change should keep its proposal, design, tasks, verification, and retrospective together. The archive is evidence, not long-term memory.

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

1. archived change or retrospective exists -> `archive`
2. deploy record or release handoff exists -> `deploy_or_done`
3. review notes or review fix commits exist -> `review`
4. verification output exists -> `verification`
5. code, prompt, or skill changes exist for this task -> `implementation`
6. technical plan or tasks exist -> `technical_planning`
7. `planning/prd.json`, `.planning/prd.json`, or change-local `prd.json` exists -> `prd_json_ready`
8. confirmed `planning/prd.html`, `.planning/prd.html`, or change-local `prd.html` exists -> `prd_confirmation`
9. draft `planning/prd.html`, `.planning/prd.html`, or change-local `prd.html` exists -> `prd_html_review`
10. requirements notes exist -> `prd_html_draft`
11. none -> `demand_discovery`

## Stage Routing

| Stage | Default Action | Companion Skill / Layer | Main Artifact | Boundary |
|---|---|---|---|---|
| `demand_discovery` | clarify business goal, users, scope, constraints, and non-goals | `demand-discovery`, `idea-exploration` | discovery brief | keep asking until the problem is bounded |
| `prd_html_draft` | create the readable PRD page | `pm-spec` | `PRD.html` | do not create `PRD.json` |
| `prd_html_review` | wait for user comments or approval | human decision gate | confirmed `PRD.html` | no silent continuation |
| `prd_confirmation` | apply requested edits until approved | `pm-spec` | confirmed `PRD.html` | record approval before structuring |
| `prd_json_ready` | produce implementation-facing structure only after approval | `pm-spec` + guardrail | `PRD.json` | source must be confirmed PRD.html |
| `technical_planning` | split modules, data flow, permissions, tests, rollback | subagents if useful | design/tasks | no broad coding yet |
| `implementation` | route to Codex/Claude, serial handoff, or project agents | `serial-agent-handoff`, native subagents | code/prompt/skill diff | Studio routes; it is not itself the executor |
| `verification` | run tests, lint, typecheck, Playwright, evals, or manual checks | command/headless + skill judgment | `verify.md` | headless proves; humans approve |
| `review` | prioritize bugs, regressions, security/data risks, missing tests | review skill/subagent | review notes and fixes | P0/P1 must be fixed or explicitly deferred |
| `deploy_or_done` | deploy only when authorized, otherwise produce handoff | project deploy flow or handoff command | release/handoff doc | public deploy/push/tag needs explicit instruction |
| `archive` | record outcome, evidence, decisions, and lessons | retrospective | archived change | only durable lessons go to memory |

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
- State files track the current workflow stage; memory stores only stable reusable rules; archives store delivery evidence.
<!-- STUDIO:END -->
```

## State, Memory, and Archive

Do not mix these three layers:

| Layer | Purpose | Examples | Lifetime |
|---|---|---|---|
| State machine | Current operational progress | current stage, artifact paths, approval flag, blocker, next allowed actions | one demand/change |
| Memory | Stable reusable context | user preferences, project rules, repeated business constraints | cross-demand |
| Archive | Audit and learning record | final PRD, tasks, verification evidence, review notes, retrospective | historical evidence |

After context compaction or session resume, trust state files and artifacts over chat memory.

## Testing, Eval-First, and Headless

For code behavior changes, prefer test-first when behavior is objectively testable:

1. write or update a failing test;
2. run it and confirm the expected failure;
3. implement the smallest change;
4. rerun and confirm pass;
5. refactor only while tests stay green.

For skill, prompt, or agent-workflow changes, use eval-first:

1. write a realistic scenario that exposes the current failure or ambiguity;
2. capture the baseline behavior;
3. revise the skill/prompt/workflow text;
4. rerun or reason through the scenario;
5. record the improvement and remaining gaps.

Headless mode is for repeatable verification only:

- schema validation;
- eval execution;
- lint, typecheck, unit tests, integration tests;
- Playwright or browser checks;
- sensitive-information scans;
- artifact consistency checks.

Headless mode must not approve business intent, hiring decisions, publishing decisions, or irreversible operations.

## Optional Commands and Hooks

The Studio skill can define command-shaped entry points, but they are optional and must preserve the same gates:

| Command | Purpose |
|---|---|
| `studio status` | report current stage, evidence, blocker, and next allowed action |
| `studio prd-html` | create or revise PRD.html only |
| `studio prd-json` | create PRD.json only after confirmed PRD.html |
| `studio verify` | run repeatable checks and write verification evidence |
| `studio handoff` | produce deploy or operator handoff without assuming deploy authority |

Hooks, when present, should be narrow guardrails:

- block `PRD.json` generation when no confirmed `PRD.html` exists;
- scan public bundles for secrets, local paths, and private/internal terms before publish;
- list hook risks and rollback instructions before installation.

Do not install hooks automatically.

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

## Red Flags

Pause and return to the previous stage when you see:

- jumping from a vague idea directly to code;
- a polished PRD.html treated as approved without explicit confirmation;
- PRD.json created in the same step as PRD.html;
- agents, MCP, hooks, or headless mode added because they sound advanced rather than because the workflow needs them;
- one broad prompt copied for every business line instead of shared rubric plus business/job configuration;
- verification claimed without command, scenario, screenshot, log, or diff evidence;
- state-machine data written into long-term memory;
- headless output treated as business approval;
- public release without a sensitive-information scan.

## Verification Report

Every verification report should include:

- artifact or code scope checked;
- command, test, or scenario used;
- pass/fail result;
- evidence path, screenshot, log, or diff;
- unresolved warnings;
- whether human approval is still required.

## Detailed Reference

Read `references/studio-pipeline.md` when executing a concrete stage or when the user asks how the whole flow should work.
