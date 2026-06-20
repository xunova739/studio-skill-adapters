---
name: autonomous-studio-codex
description: Codex version of the Studio workflow router. Use when the user mentions studio, autonomous mode, next step, demand-to-launch, PRD.html, PRD.json after approval, agent/skill development, verification, review, handoff, or asks what stage a project is in from Codex/Codex Desktop. Do not use for simple Q&A, typo edits, direct bug fixes, or one-shot research unless the user explicitly asks to enter Studio. Project-local version using AGENTS.md and planning/status.json/.planning/status.json; does not install Claude hooks or depend on Claude model names/CronCreate.
---

# Autonomous Studio for Codex

This is the Codex install form of Autonomous Studio. It runs the Studio workflow through Codex-safe project files and the current Codex session.

Core principle: Studio turns unclear intent into traceable artifacts. AI may draft, inspect, implement, and verify; humans approve intent, business rules, publishing, irreversible actions, and high-impact decisions.

## 适配边界

- 使用项目本地 `.codex/skills/autonomous-studio-codex`。
- 读取并维护短规则块到 `AGENTS.md`，不要把长流程、教程、路径清单写进根规则文件。
- 优先读取现有 `planning/status.json`，没有再读取 `.planning/status.json`。不要同时创建两套状态目录。
- 不安装、不执行 Claude Code hooks；不写入 `~/.claude`。
- 不使用 Claude 模型名或 CronCreate 作为 Codex 执行前提。
- 可以使用 Codex native subagents，但只在任务有独立、边界清楚、可验证的子 lanes 时使用。
- Git 操作遵循当前项目 `AGENTS.md` / 用户指令；没有明确要求时不要自动 push。
- Studio 负责阶段路由和门槛，不替代 Codex 作为开发执行器。
- 状态机记录当前工作进度；Memory 只保存稳定可复用规则；Archive 保存交付证据。

## 启动流程

1. 判断当前项目是否已有状态文件：
   - 首选 `planning/status.json`
   - 其次 `.planning/status.json`
   - 都没有时，根据已有文档和用户目标临时推断阶段，不强制创建文件
2. 用一句话报告当前阶段、依据和下一步。
3. 如果用户要求“开启/注入 Studio”，只向 `AGENTS.md` 写入下面的短块，并保持 marker 可替换。

```markdown
<!-- STUDIO:LOCAL-CODEX:BEGIN -->
# Studio Local Workflow (Codex)

- State: prefer `planning/status.json`; fallback `.planning/status.json`.
- Flow: discovery -> PRD.html -> human confirmation -> PRD.json -> technical plan -> implementation -> verification -> review -> deploy handoff.
- `PRD.html` is a separate review artifact. Do not write `PRD.json` until the user confirms it.
- Codex owns execution in the current session; Claude hooks and background cron are not required.
- Keep AI辅助、人主导: every AI decision must expose evidence, score/reason, and human override path.
- State is workflow progress, not memory. Archive final PRD/tasks/verification/retrospective as evidence.
- Do not auto-push unless the current project policy or user explicitly asks for it.
<!-- STUDIO:LOCAL-CODEX:END -->
```

## 阶段路由

根据状态和用户语义选择最小必要动作：

- `discovery`: 用 `demand-discovery` 或 `idea-exploration` 澄清问题、角色、边界、MVP。
- `prd_html`: 用 `pm-spec` 写 `PRD.html`，先作为人工确认页，不同时生成 `PRD.json`。
- `prd_review`: 等用户明确确认或要求修改；"先看看/大概可以/继续聊"不算确认。
- `prd_json`: 用户明确确认 `PRD.html` 后，再生成结构化 `PRD.json`。
- `technical_plan`: 产出实现拆解、数据结构、接口、验证计划。复杂时先规划，不直接改代码。
- `implementation`: 由 Codex 直接执行或交给 `serial-agent-handoff`；需要多 lane 时使用 native subagents。不要依赖 Claude Code Agent。
- `verification`: 运行项目已有测试、类型检查、lint、Playwright、evals、敏感信息扫描或人工验证清单。
- `review`: 用代码审查姿态输出问题、风险、缺测试点；必要时再修。
- `deploy`: 只在项目已有部署权限和流程时执行；否则交付部署清单或交给项目自己的部署流程。公开 push/tag/release 需要明确祈使句授权。
- `archive`: 写 `retrospective.md`，归档最终 artifact、验证证据和人工决策。

## Change Folder

非简单任务优先把产物放在同一个变更目录：

```text
planning/changes/<change-id>/
  prd.html
  prd.json
  design.md
  tasks.md
  verify.md
  retrospective.md
```

如果项目已有 `.planning/changes/`，沿用项目约定。完成后可以归档到 `planning/changes/archive/YYYY-MM-DD-<change-id>/`。

## 状态文件建议

如果需要创建状态文件，使用已有项目风格；没有风格时用：

```json
{
  "stage": "discovery",
  "last_updated": "YYYY-MM-DD",
  "current_goal": "",
  "artifacts": [],
  "human_decisions": [],
  "blocked_reason": null,
  "next_allowed_actions": [],
  "next_action": ""
}
```

`stage` 只能使用：`discovery`、`prd_html`、`prd_review`、`prd_json`、`technical_plan`、`implementation`、`verification`、`review`、`deploy`、`archive`、`done`。

## TDD / Eval-first / Headless

- 代码行为变化：优先测试先行。先写失败测试，确认按预期失败，再写最小实现并确认通过。
- Skill、Prompt、Agent、流程规则变化：使用 eval-first。先写真实失败场景和基线，再改文本并记录改进。
- Headless 只用于可重复验证：schema、evals、lint、typecheck、tests、Playwright、敏感信息扫描、artifact 一致性。
- Headless 不批准业务意图、招聘筛选结论、公开发布、部署或不可逆操作。

## 质量要求

- 明确区分“AI 判断”和“规则/人工确认”。
- 对每个自动化建议给出依据字段，例如来源简历、岗位要求、评分规则、历史通过样本。
- 不把提示词散落到每个岗位；优先做业务线配置、岗位配置、评分维度和少量可复用 prompt 模板。
- 任何涉及候选人筛选、淘汰、推荐排序的功能，都要保留人工 override 和审核记录。
- 如果目标环境离线或内网运行，PRD/实现必须写清楚离线可用方式。
- 验证报告必须包含范围、命令/场景、通过/失败、证据路径、未解决 warning、是否仍需人工确认。
- 不把状态机写入长期 memory；只把稳定偏好、项目规则、可复用经验沉淀为 memory。
