---
name: autonomous-studio-codex
description: Codex 版 Studio 研发流程路由器。当用户在 Codex/Codex Desktop 中提到 studio、自主研发、自动继续、从需求到上线、需求/PRD/开发/验证/评审/部署串联、或询问当前项目下一步该做什么时使用。项目本地版，使用 AGENTS.md 和 planning/status.json/.planning/status.json；不安装 Claude hooks，不写 ~/.claude，不依赖 sonnet/opus/CronCreate。
---

# Autonomous Studio for Codex

本 skill 是 `autonomous-studio` 的 Codex 本地适配壳。它保留原 Studio 的“状态驱动研发流程”思想，但把运行面收敛到 Codex 可控的项目本地文件和当前会话。

## 适配边界

- 使用项目本地 `.codex/skills/autonomous-studio-codex`，不要把旧 bundle 整包复制进来。
- 读取并维护短规则块到 `AGENTS.md`，不要把长流程、教程、路径清单写进根规则文件。
- 优先读取现有 `planning/status.json`，没有再读取 `.planning/status.json`。不要同时创建两套状态目录。
- 不安装、不执行 Claude Code hooks；不写入 `~/.claude`。
- 不使用 Claude 模型名、CronCreate 或旧三层心跳作为 Codex 执行前提。
- 可以使用 Codex native subagents，但只在任务有独立、边界清楚、可验证的子 lanes 时使用。
- Git 操作遵循当前项目 `AGENTS.md` / 用户指令；没有明确要求时不要自动 push。

## 来源仓库配置

原 Studio skill 开头如果有内部仓库或 `sync.target_repo`，在 Codex 本地版中替换为你们上传到 GitHub 的镜像仓库配置。规则是：仓库 URL 只能是无凭据 URL，认证交给 `gh`、SSH key 或系统 credential helper。

```yaml
studio_source:
  kind: github-mirror
  repo: "https://github.com/<org>/<repo>.git"
  branch: "main"
  credential_rule: "Do not embed token, password, authTicket, cookie, or private key in SKILL.md."
```

如果 GitHub 在当前内网不可访问，仍然保留这个配置作为“来源说明”，不要让 skill 运行依赖实时访问 GitHub。

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
- Source: use the configured GitHub mirror as reference only; never embed credentials in repo URLs.
- Flow: discovery -> PRD -> technical plan -> implementation -> verification -> review -> deploy handoff.
- Codex owns execution in the current session; Claude hooks and background cron are not required.
- Keep AI辅助、人主导: every AI decision must expose evidence, score/reason, and human override path.
- Do not auto-push unless the current project policy or user explicitly asks for it.
<!-- STUDIO:LOCAL-CODEX:END -->
```

## 阶段路由

根据状态和用户语义选择最小必要动作：

- `discovery`: 用 `demand-discovery` 或 `idea-exploration` 澄清问题、角色、边界、MVP。
- `prd`: 用 `pm-spec` 写 PRD，把验收口径、状态流、人工确认点写清楚。
- `technical_plan`: 产出实现拆解、数据结构、接口、验证计划。复杂时先规划，不直接改代码。
- `implementation`: 由 Codex 直接执行；需要多 lane 时使用 native subagents。不要依赖 Claude Code Agent。
- `verification`: 运行项目已有测试、类型检查、lint、Playwright 或人工验证清单。
- `review`: 用代码审查姿态输出问题、风险、缺测试点；必要时再修。
- `deploy`: 只在项目已有部署权限和流程时调用 `prod-deploy` 或交付部署清单。

## 状态文件建议

如果需要创建状态文件，使用已有项目风格；没有风格时用：

```json
{
  "stage": "discovery",
  "last_updated": "YYYY-MM-DD",
  "current_goal": "",
  "artifacts": [],
  "human_decisions": [],
  "next_action": ""
}
```

`stage` 只能使用：`discovery`、`prd`、`technical_plan`、`implementation`、`verification`、`review`、`deploy`、`done`。

## 质量要求

- 明确区分“AI 判断”和“规则/人工确认”。
- 对每个自动化建议给出依据字段，例如来源简历、岗位要求、评分规则、历史通过样本。
- 不把提示词散落到每个岗位；优先做业务线配置、岗位配置、评分维度和少量可复用 prompt 模板。
- 任何涉及候选人筛选、淘汰、推荐排序的功能，都要保留人工 override 和审核记录。
- 内网项目里，外部 GitHub 项目只能作为参考来源；PRD/实现必须写清楚离线可用方式。
