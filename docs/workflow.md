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

| Stage | User Action | System / Agent Response | Main Artifact | Exit Criteria |
|---|---|---|---|---|
| 1. Demand Discovery | Explain the business goal, users, current pain, constraints, and examples. | Ask focused questions, reduce ambiguity, define MVP and non-goals. | Demand notes or discovery brief | Problem, users, value, scope, and risks are clear enough to write a PRD. |
| 2. PRD.html Draft | Explicitly ask for `PRD.html`, or trigger the PM/spec skill in HTML mode. | Convert the clarified demand into a visual/readable product contract. Do not create `PRD.json` in this step. | `PRD.html` | Stakeholders can review the product behavior without reading implementation code. |
| 3. PRD Confirmation | Review the `PRD.html` and explicitly approve or request changes. | Apply edits until the product contract is accepted. | Confirmed `PRD.html` | User/stakeholder confirmation is recorded. |
| 4. PRD.json Structuring | After confirmation, ask to write the structured PRD. | Convert the confirmed `PRD.html` into implementation-facing structured data. | `PRD.json` | Core fields, state flow, acceptance checks, and human decision points are machine-readable. |
| 5. Technical Planning | Confirm that `PRD.json` is ready for implementation planning. | Split modules, data flow, APIs, permissions, test strategy, and rollback boundaries. | Technical plan and task list | Each implementation task has owner surface, file scope, and verification method. |
| 6. Agent Skill Development | Invoke the relevant development skill or agent lane. | Implement in small steps, keep status visible, avoid unrelated refactors. | Code diff, config, scripts, docs | The requested behavior exists locally and can be verified. |
| 7. Verification | Ask the agent to prove it works. | Run lint, typecheck, tests, Playwright/manual checks, or scenario replay as appropriate. | Test output and verification notes | The claim is backed by concrete evidence or the remaining risk is explicit. |
| 8. Review and Fix | Ask for review before release. | Prioritize bugs, regressions, missing tests, security/data risks, and UX issues. | Review notes and follow-up diffs | P0/P1 risks are resolved or consciously deferred. |
| 9. Deploy / Handoff | Ask for deployment or handoff. | Use the project deployment flow, or produce a handoff checklist when deployment is not authorized. | Release notes, deployment record, handoff doc | The next operator can continue without hidden context. |
| 10. Feedback Loop | Bring back user feedback, defects, or new business-line needs. | Update prompt templates, business-line config, scoring dimensions, and reusable skill rules. | Changelog, prompt/config updates, next backlog | The next cycle starts from evidence, not from scratch. |

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

## Agent Development Rule

After `PRD.html`, do not jump straight into broad coding. Use a development agent or skill only after these are clear:

- what file/module surface can change;
- what behavior must stay unchanged;
- what command or scenario proves success;
- what human decision remains outside AI automation.

For multi-business-line systems, prefer shared templates and business-line configuration over one prompt per job. A useful split is:

```text
global screening rubric
  + business-line policy/config
  + job-specific requirements
  + evidence extraction
  + human review and override
```

This avoids spending half a day tuning a separate prompt for every position.
