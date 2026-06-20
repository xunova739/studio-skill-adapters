# Studio Pipeline Reference

This reference expands the stage behavior used by `SKILL.md`.

## Operating Model

Studio is a workflow router, not a black-box executor. It keeps three records separate:

| Record | Purpose | Do Not Use It For |
|---|---|---|
| State machine | current stage, approvals, blockers, next allowed actions | long-term preference storage |
| Memory | stable reusable user/project/business rules | temporary task progress |
| Archive | final artifacts, verification evidence, review notes, retrospective | runtime gating by itself |

When a session resumes after compaction, read state files and artifacts before trusting the conversation transcript.

For substantial work, create or reuse one change folder:

```text
planning/changes/<change-id>/
  prd.html
  prd.json
  design.md
  tasks.md
  verify.md
  retrospective.md
```

Use `.planning/changes/` if the project already uses `.planning/`. Archive completed changes under `planning/changes/archive/YYYY-MM-DD-<change-id>/`.

## 1. Demand Discovery

Goal: turn a rough idea into a bounded demand.

Default companion skills:

- `demand-discovery` for structured product/workflow requirement discovery;
- `idea-exploration` for early unclear ideas, feasibility, and option shaping.

Capture:

- business goal;
- target users or business lines;
- current pain;
- constraints;
- non-goals;
- examples;
- risks and human approval points.

Exit when the problem, value, scope, and risks are clear enough to draft `PRD.html`.

Do not create implementation tasks in this stage.

## 2. PRD.html Draft

`PRD.html` is the human-readable review artifact. It can be triggered independently.

It should show:

- user roles;
- business-line differences;
- workflow and state transitions;
- UI or page behavior;
- AI-vs-human decision boundaries;
- webhook or notification routing;
- acceptance criteria;
- known risks and non-goals.

Do not create `PRD.json` in this stage.

`PRD.html` should separate:

- confirmed facts;
- assumptions;
- open questions;
- proposed scope;
- explicit human decision points.

## 3. PRD Confirmation

Wait for the user to confirm, annotate, or request changes to `PRD.html`.

Only after confirmation can the workflow proceed to `PRD.json`.

Valid confirmation examples:

- "approved, generate PRD.json";
- "确认了，继续结构化";
- "按这个 PRD.html 写执行用 JSON";
- a revision request followed by explicit acceptance.

Invalid confirmation examples:

- "先看看";
- "大概可以";
- "继续聊";
- silence or no response.

## 4. PRD.json Structuring

`PRD.json` is the implementation-facing contract.

Recommended structure:

```json
{
  "version": "1.0",
  "source": "planning/PRD.html",
  "confirmed": true,
  "nodes": [
    {
      "id": "node-1",
      "name": "Workflow node",
      "tasks": [
        {
          "id": "N1-01",
          "title": "Task title",
          "description": "Implementation-ready detail",
          "acceptance": ["Specific, testable condition"],
          "priority": "P0",
          "status": "pending",
          "completedAt": null
        }
      ]
    }
  ],
  "metadata": {
    "humanDecisionPoints": [],
    "stateMachine": {}
  }
}
```

Rules:

- every task needs a unique id;
- descriptions must be detailed enough for an agent to implement;
- acceptance criteria must be testable;
- AI decisions must include evidence and override path;
- P0 completion should drive verification, not deployment.
- `source` must point to the confirmed `PRD.html`;
- `confirmed` must not be true unless a human approval decision is recorded.

## 5. Technical Planning

Split the confirmed `PRD.json` into:

- modules or files likely to change;
- APIs and data flow;
- permissions and audit requirements;
- tests and manual checks;
- rollback or handoff boundaries.

Do not start broad coding until the success check is explicit.

The plan should state whether each task uses:

- TDD for code behavior changes;
- eval-first for skill/prompt/agent changes;
- manual proof for subjective business review;
- headless proof for repeatable checks.

If a task spans independent files, modules, or risk lanes, consider subagents. If the task touches one tightly coupled surface, keep it inline.

## 6. Agent Skill Development

Use the relevant development skill or agent lane.

Studio chooses the lane; Codex, Claude Code, project agents, or `serial-agent-handoff` execute it.

Rules:

- keep edits narrow;
- avoid unrelated refactors;
- keep progress visible;
- update task status when the project uses status files;
- stop for destructive, external-production, credential-gated, or materially branching actions.
- use TDD when behavior can be tested;
- use eval-first when changing skills, prompts, routing rules, or workflow instructions.

### TDD for Code

1. Write or update one failing test for the intended behavior.
2. Run it and confirm it fails for the expected reason.
3. Implement the smallest change that makes it pass.
4. Rerun the test and relevant surrounding checks.
5. Refactor only while tests remain green.

### Eval-First for Skills and Prompts

1. Write a realistic scenario that exposes the current failure or ambiguity.
2. Record the baseline behavior or expected failure.
3. Revise the skill, prompt, or workflow text.
4. Rerun or dry-run the scenario.
5. Record the improvement and any remaining uncovered cases.

## 7. Verification

Use the lightest proof that actually validates the claim:

- lint/typecheck/unit tests;
- Playwright or E2E tests;
- evals for skill/prompt behavior;
- schema validation for structured artifacts;
- sensitive-information scan before public release;
- scenario replay;
- manual checklist;
- screenshots or logs where useful.

If verification fails, fix and rerun the relevant check instead of reporting success.

Headless can run repeatable checks, but it cannot approve business intent, hiring decisions, publication, deployment, or irreversible operations.

Verification report format:

```markdown
## Verification

- Scope checked:
- Command/test/scenario:
- Result: pass/fail
- Evidence:
- Warnings:
- Human approval still required:
```

## 8. Review and Fix

Review should lead with:

- bugs;
- regressions;
- missing tests;
- security or data risks;
- unclear AI/human decision boundaries;
- UX or workflow breakpoints.

Summaries come after findings.

## 9. Deploy or Handoff

Deploy only when the user or project policy explicitly allows it.

When deployment is not authorized, produce a handoff:

- what changed;
- what was verified;
- what remains risky;
- how to deploy;
- rollback notes.

Public deploy, push to default branch, tag, release, or external notification requires explicit instruction. A user question like "is it ready?" is not authorization.

## 10. Feedback Loop

Feed real usage back into:

- prompt templates;
- business-line config;
- scoring dimensions;
- test cases;
- reusable skill rules.

## 11. Archive and Retrospective

Write a retrospective when a non-trivial change completes:

```markdown
# Retrospective

## Goal
## Final Artifacts
## What Changed
## Verification Evidence
## Human Decisions
## Risks / Deferred Work
## Durable Lessons
```

Only the durable lessons belong in memory. Keep temporary stage progress and local file paths in the state/archive layer instead.

## Optional Commands and Hooks

Command-shaped entries can be documented before implementation:

- `studio status`: current stage, evidence, blocker, next action;
- `studio prd-html`: PRD.html only;
- `studio prd-json`: confirmed PRD.html to PRD.json;
- `studio verify`: repeatable checks and evidence;
- `studio handoff`: deployment/operator handoff.

Optional hooks should only guard boundaries:

- block PRD.json before confirmed PRD.html;
- scan public bundles before publish;
- require hook risk and rollback summary before installing hooks.

Do not use hooks to make business decisions.
