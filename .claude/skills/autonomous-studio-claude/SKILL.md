---
name: autonomous-studio-claude
description: Claude Code version of the Studio workflow router. Use when the user mentions studio, autonomous mode, next step, demand-to-launch, PRD.html, PRD.json after approval, agent/skill development, verification, review, handoff, or asks what stage a project is in from Claude Code. Do not use for simple Q&A, typo edits, direct bug fixes, or one-shot research unless the user explicitly asks to enter Studio. Project-local version using CLAUDE.md; hooks are optional and require review.
---

# Autonomous Studio for Claude Code

This is the Claude Code install form of Autonomous Studio. It allows a Claude Code project to use the Studio stage router while keeping hooks optional and reviewable.

Core principle: Studio turns unclear intent into traceable artifacts. AI may draft, inspect, implement, and verify; humans approve intent, business rules, publishing, irreversible actions, and high-impact decisions.

## 适配边界

- 使用项目本地 `.claude/skills/autonomous-studio-claude`。
- 面向 Claude Code 时写入 `CLAUDE.md`，不要写入 `AGENTS.md`。
- 可以兼容 Claude Code hooks、Agent、后台心跳，但必须先经过人工确认和安全审查。
- 不复制 `.claude/sessions`、decision logs、checkpoints、heartbeat、memory dump。
- 不复制任何带 token、password、authTicket、cookie、API key、私钥或用户名密码的仓库 URL。
- Studio 负责阶段路由和门槛，不替代 Claude Code 作为开发执行器。
- 状态机记录当前工作进度；Memory 只保存稳定可复用规则；Archive 保存交付证据。

## 启动流程

1. 判断当前项目状态：
   - 首选 `.planning/status.json`
   - 其次 `planning/status.json`
   - 都没有时，根据已有 PRD、任务文档、代码变更临时推断阶段
2. 如果用户要求“开启/注入 Studio”，向 `CLAUDE.md` 写入短 marker 块。
3. 如果用户要求安装 hooks，先列出将安装的 hook 文件、触发时机、可回滚方式和安全风险，再执行。

```markdown
<!-- STUDIO:LOCAL-CLAUDE:BEGIN -->
# Studio Local Workflow (Claude Code)

- State: prefer `.planning/status.json`; fallback `planning/status.json`.
- Flow: discovery -> PRD.html -> human confirmation -> PRD.json -> technical plan -> implementation -> verification -> review -> deploy handoff.
- `PRD.html` is a separate review artifact. Do not write `PRD.json` until the user confirms it.
- Hooks are optional and require explicit review before installation.
- Keep AI辅助、人主导: every AI decision must expose evidence, score/reason, and human override path.
- State is workflow progress, not memory. Archive final PRD/tasks/verification/retrospective as evidence.
- Do not copy sessions, decision logs, checkpoints, memory dumps, or credential files.
<!-- STUDIO:LOCAL-CLAUDE:END -->
```

## 阶段路由

- `discovery`: 澄清目标、业务线、角色、MVP、人机边界。
- `prd_html`: 产出 `PRD.html`、验收标准、人工确认点、离线可用要求；等待确认。
- `prd_review`: 等用户明确确认或要求修改；"先看看/大概可以/继续聊"不算确认。
- `prd_json`: 用户明确确认 `PRD.html` 后，产出结构化 `PRD.json`。
- `technical_plan`: 产出架构、数据流、接口、测试方案。
- `implementation`: 可以使用 Claude Code Agent 或 `serial-agent-handoff`；并发前先拆清文件/模块边界。
- `verification`: 运行测试、类型检查、lint、Playwright、evals、敏感信息扫描或人工验证清单。
- `review`: 代码审查优先列风险、缺陷、缺测试。
- `deploy`: 只在权限和流程明确时执行，否则产出部署交接清单。公开 push/tag/release 需要明确祈使句授权。
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

## Hook 安装规则

只有用户明确要求“安装 Claude hooks”时才执行安装。安装前必须确认：

- hook 文件必须来自已审查的本地文件。
- hook 不包含外部凭据、内部 token、硬编码 webhook 或不可解释的网络请求。
- hook 写入范围只在当前项目 `.claude/` 内，除非用户明确授权全局安装。
- 每个 hook 都有回滚办法。
- hook 只能守边界，例如未确认 PRD.html 不写 PRD.json、发布前敏感扫描、安装前风险提示。
- hook 不批准业务意图、招聘筛选结论、发布或部署。

## TDD / Eval-first / Headless

- 代码行为变化：优先测试先行。先写失败测试，确认按预期失败，再写最小实现并确认通过。
- Skill、Prompt、Agent、流程规则变化：使用 eval-first。先写真实失败场景和基线，再改文本并记录改进。
- Headless 只用于可重复验证：schema、evals、lint、typecheck、tests、Playwright、敏感信息扫描、artifact 一致性。
- Headless 不批准业务意图、招聘筛选结论、公开发布、部署或不可逆操作。

## 质量要求

- 根 `CLAUDE.md` 保持短规则和路由，不承载长教程。
- 长流程放在 skill、README 或 docs。
- AI 输出必须保留依据、评分解释、人工 override 和审核记录。
- 对招聘、候选人筛选、岗位匹配等高影响场景，默认“AI 辅助，人工主导”，不要做黑盒淘汰。
- 验证报告必须包含范围、命令/场景、通过/失败、证据路径、未解决 warning、是否仍需人工确认。
- 不把状态机写入长期 memory；只把稳定偏好、项目规则、可复用经验沉淀为 memory。
