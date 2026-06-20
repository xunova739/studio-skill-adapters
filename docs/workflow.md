# Studio Workflow

This is the intended end-to-end flow for using the Studio adapters.

## One-Line Summary

```text
聊需求 -> 单独触发 PRD.html -> 人工确认 -> PRD.json -> 技术方案/验收拆分 -> Agent Skill 开发 -> 验证评审 -> 部署交接 -> 回流沉淀
```

The key product principle is:

```text
AI assists. Humans decide.
```

## Stage Map

| Stage | User Action | System / Agent Response | Main Artifact | Optional Automation | Exit Criteria |
|---|---|---|---|---|---|
| 1. Demand Discovery | Explain the business goal, users, current pain, constraints, and examples. | Use `demand-discovery` or `idea-exploration`; ask focused questions, reduce ambiguity, define MVP and non-goals. | Demand notes or discovery brief | Skill only; no headless gate. | Problem, users, value, scope, and risks are clear enough to write a PRD. |
| 2. PRD.html Draft | Explicitly ask for `PRD.html`, or trigger the PM/spec skill in HTML mode. | Use `pm-spec` to create a visual/readable product contract. Do not create `PRD.json` in this step. | `PRD.html` | Optional `studio prd-html` command. | Stakeholders can review product behavior without reading implementation code. |
| 3. PRD Confirmation | Review the `PRD.html` and explicitly approve or request changes. | Apply edits until the product contract is accepted. Record approval. | Confirmed `PRD.html` | Optional guardrail preventing JSON generation before approval. | User/stakeholder confirmation is recorded. |
| 4. PRD.json Structuring | After confirmation, ask to write the structured PRD. | Convert the confirmed `PRD.html` into implementation-facing structured data. | `PRD.json` | Optional schema validation. | Core fields, state flow, acceptance checks, and human decision points are machine-readable. |
| 5. Technical Planning | Confirm that `PRD.json` is ready for implementation planning. | Split modules, data flow, APIs, permissions, test strategy, and rollback boundaries. | `design.md` and `tasks.md` | Subagents for independent analysis lanes. | Each implementation task has owner surface, file scope, and verification method. |
| 6. Implementation / Skill Development | Invoke the relevant development skill or agent lane. | Route to Codex, Claude Code, `serial-agent-handoff`, or native subagents. Use TDD for code and eval-first for skills/prompts. | Code, config, script, doc, prompt, or skill diff | Subagents; TDD/eval harness. | The requested behavior exists locally and can be verified. |
| 7. Verification | Ask the agent to prove it works. | Run lint, typecheck, tests, Playwright/manual checks, evals, scenario replay, and sensitive scans as appropriate. | `verify.md` | `studio verify`, headless, CI. | The claim is backed by concrete evidence or remaining risk is explicit. |
| 8. Review and Fix | Ask for review before release. | Prioritize bugs, regressions, missing tests, security/data risks, and UX issues. | Review notes and follow-up diffs | Review subagent/skill. | P0/P1 risks are resolved or consciously deferred. |
| 9. Deploy / Handoff | Ask for deployment or handoff. | Use the project deployment flow, or produce a handoff checklist when deployment is not authorized. Public deploy/push/tag requires explicit instruction. | Release notes, deployment record, handoff doc | `studio handoff`; project-owned deploy command. | The next operator can continue without hidden context. |
| 10. Archive / Feedback Loop | Bring back user feedback, defects, or new business-line needs. | Archive artifacts, write retrospective, update prompt templates, business-line config, scoring dimensions, and reusable skill rules. | `retrospective.md`, archive folder, changelog | Optional archive command. | The next cycle starts from evidence, not from scratch. |

## Change Folder

For non-trivial work, keep artifacts together in one change folder:

```text
planning/changes/<change-id>/
  prd.html
  prd.json
  design.md
  tasks.md
  verify.md
  retrospective.md
```

Use `.planning/changes/` instead when the target project already uses `.planning/`. Completed changes can be moved under `planning/changes/archive/YYYY-MM-DD-<change-id>/`.

The folder is the delivery record. It is not long-term memory.

## Where `PRD.html` Fits

`PRD.html` is the human-readable contract and can be triggered independently. It should make product behavior visible:

- user roles and business lines;
- main workflows and state transitions;
- AI-vs-human decision boundaries;
- webhook or notification routing;
- acceptance criteria;
- known non-goals and risks.

It should not automatically produce `PRD.json`. The intended contract is:

```text
PRD.html first
  -> user confirms
  -> then PRD.json
```

`PRD.json` is created only after confirmation, so implementation does not begin from an unapproved product contract. Keep machine-readable companion notes when the project needs them, such as status JSON, test cases, API tables, or prompt/config matrices.

Confirmation must be explicit. "Looks fine", "let me see", "continue discussing", or silence is not enough unless the user also asks to proceed into structured PRD or implementation planning.

## Agent Development Rule

After `PRD.html`, do not jump straight into broad coding. Use a development agent or skill only after these are clear:

- what file/module surface can change;
- what behavior must stay unchanged;
- what command or scenario proves success;
- what human decision remains outside AI automation.
- whether code changes should use TDD;
- whether skill/prompt changes should use eval-first;
- whether a headless check can prove the claim.

Studio itself is not the implementation engine. It routes to the right execution surface:

- Codex or Claude Code for implementation in the current project;
- `serial-agent-handoff` for staged agent execution;
- native subagents for independent, bounded lanes;
- `agent-context-authoring` or `agents-map` for context structure;
- `excalidraw-diagram-skill` when a workflow/architecture diagram is the useful artifact.

For multi-business-line systems, prefer shared templates and business-line configuration over one prompt per job. A useful split is:

```text
global screening rubric
  + business-line policy/config
  + job-specific requirements
  + evidence extraction
  + human review and override
```

This avoids spending half a day tuning a separate prompt for every position.

## Testing and Headless

Use TDD for code behavior when the behavior is objectively testable:

```text
write failing test -> watch expected failure -> implement minimal code -> watch pass -> refactor while green
```

Use eval-first for skills, prompts, agent instructions, and workflow rules:

```text
write realistic failure scenario -> record baseline -> revise instruction -> rerun scenario -> record improvement
```

Use headless only for repeatable verification:

- schema validation;
- eval execution;
- lint/typecheck/tests;
- Playwright or browser checks;
- sensitive-information scans;
- artifact consistency checks.

Headless does not approve business intent, recruiting decisions, publishing, deployment, or irreversible operations.

## State, Memory, Archive

| Layer | What It Stores | Example |
|---|---|---|
| State machine | current workflow position | current stage, artifact path, confirmation flag, blocker, next action |
| Memory | stable reusable context | user preference, project rule, recurring business constraint |
| Archive | completed delivery evidence | PRD, design, tasks, verify log, review result, retrospective |

After context compaction, trust state files and artifacts over chat memory.
