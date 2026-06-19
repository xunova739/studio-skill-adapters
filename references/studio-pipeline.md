# Studio Pipeline Reference

This reference expands the stage behavior used by `SKILL.md`.

## 1. Demand Discovery

Goal: turn a rough idea into a bounded demand.

Capture:

- business goal;
- target users or business lines;
- current pain;
- constraints;
- non-goals;
- examples;
- risks and human approval points.

Exit when the problem, value, scope, and risks are clear enough to draft `PRD.html`.

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

## 3. PRD Confirmation

Wait for the user to confirm, annotate, or request changes to `PRD.html`.

Only after confirmation can the workflow proceed to `PRD.json`.

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

## 5. Technical Planning

Split the confirmed `PRD.json` into:

- modules or files likely to change;
- APIs and data flow;
- permissions and audit requirements;
- tests and manual checks;
- rollback or handoff boundaries.

Do not start broad coding until the success check is explicit.

## 6. Agent Skill Development

Use the relevant development skill or agent lane.

Rules:

- keep edits narrow;
- avoid unrelated refactors;
- keep progress visible;
- update task status when the project uses status files;
- stop for destructive, external-production, credential-gated, or materially branching actions.

## 7. Verification

Use the lightest proof that actually validates the claim:

- lint/typecheck/unit tests;
- Playwright or E2E tests;
- scenario replay;
- manual checklist;
- screenshots or logs where useful.

If verification fails, fix and rerun the relevant check instead of reporting success.

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

## 10. Feedback Loop

Feed real usage back into:

- prompt templates;
- business-line config;
- scoring dimensions;
- test cases;
- reusable skill rules.
