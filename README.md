# Studio Skill Adapters

把一套 Studio 研发流程，拆成两种本地可安装的 Skill：

- **Codex 版**：给 Codex / Codex Desktop 用，入口是 `AGENTS.md`。
- **Claude Code 版**：给 Claude Code 用，入口是 `CLAUDE.md`。

这个仓库不是完整研发框架，也不是自动部署系统。它只做一件事：把原来混在一起的 Studio workflow，整理成可审查、可复制、不会携带内部凭据的本地 skill 适配层。

## 你什么时候用它

- 你已经有一套“聊需求 -> 写 PRD -> Agent 开发 -> 验证评审”的工作流。
- 你希望同一套流程既能在 Codex 用，也能在 Claude Code 用。
- 你不想整包复制旧 Studio bundle，因为里面可能混着 hooks、历史会话、内部仓库地址、同步配置或本地路径。
- 你要把 skill 上传 GitHub，但不能泄露 token、内部 URL、cookie、私钥或旧 runtime 状态。

## 标准流程

```text
聊需求
  -> 单独触发 PRD.html
  -> 人工确认 PRD.html
  -> 再写 PRD.json
  -> 技术方案 / 验收拆分
  -> Agent Skill 开发
  -> 验证 / 评审 / 修复
  -> 部署交接
  -> 回流沉淀
```

关键边界：

- `PRD.html` 是给人看的确认页，可以单独触发。
- `PRD.json` 是给实现和 Agent 执行用的结构化合同。
- 不会在生成 `PRD.html` 时顺手生成 `PRD.json`。
- 只有人工确认 `PRD.html` 后，才进入 `PRD.json`。

详细流程见 [docs/workflow.md](docs/workflow.md)。

## 这里每个东西是干嘛的

| 路径 | 用途 |
|---|---|
| `.codex/skills/autonomous-studio-codex/` | Codex 本地 skill。负责读取项目状态、路由到需求/PRD/开发/验证等阶段，并只向 `AGENTS.md` 注入短规则块。 |
| `.claude/skills/autonomous-studio-claude/` | Claude Code 本地 skill。负责同一套 Studio 流程，但面向 `CLAUDE.md`；hooks 只作为可选项，不能默认安装。 |
| `docs/workflow.md` | 端到端流程说明，重点写清 `PRD.html` 与 `PRD.json` 的先后关系。 |
| `docs/luban-showcase.md` | 鲁班打磨记录：定位、评分、发布缺口和出师卡。它是评估文档，不是用户安装入口。 |
| `examples/prompts.md` | 可以直接复制试用的触发语。 |
| `examples/status.example.json` | Studio 状态文件样例。 |
| `SECURITY.md` | 公开发布时的安全边界。 |
| `ALIASES.md` | 推荐在长期文档里使用的 skill 别名。 |

## 安装方式

复制你需要的那一套到目标项目。

```bash
git clone https://github.com/xunova739/studio-skill-adapters.git
cd studio-skill-adapters

# 安装到 Codex 项目
mkdir -p /path/to/project/.codex/skills
cp -R .codex/skills/autonomous-studio-codex /path/to/project/.codex/skills/

# 安装到 Claude Code 项目
mkdir -p /path/to/project/.claude/skills
cp -R .claude/skills/autonomous-studio-claude /path/to/project/.claude/skills/
```

如果 Codex 或 Claude Code 缓存了 skill 列表，复制后重载项目或重启客户端。

## 触发方式

可以这样说：

- `studio，帮我判断这个项目现在处于哪个阶段`
- `开启本地 Studio，但不要装 Claude hooks`
- `单独生成 PRD.html，先不要写 PRD.json`
- `PRD.html 已确认，现在写 PRD.json`
- `从需求到上线，按 Studio 流程推进`
- `检查这个 skill 有没有把 token 或内部 URL 带出去`

## GitHub Mirror 配置

旧 skill 里如果有内部 `target_repo` 或 sync block，公开发布时替换成无凭据的 GitHub mirror：

```yaml
studio_source:
  kind: github-mirror
  repo: "https://github.com/<org>/<repo>.git"
  branch: "main"
  credential_rule: "Do not embed credentials in skill files."
```

认证交给 `gh auth`、SSH key 或系统 credential helper。不要把 token、cookie、密码、私钥、authTicket 写进 skill 文件。

## 安全边界

这两套 adapter 默认不会：

- 自动安装 Claude hooks；
- 复制旧 sessions、decision logs、checkpoints、memory dump；
- 在仓库 URL 里嵌入凭据；
- 未经项目规则或用户明确要求就 push / deploy；
- 对招聘筛选、候选人排序等高影响场景做黑盒决策。

AI 的定位是辅助，不是替人拍板。涉及候选人筛选、业务线配置、岗位匹配时，应保留依据、评分原因、人工复核和 override。

## 发布前验证

当前发布前已检查：

- `jq` 可以解析两个 eval 文件和示例状态 JSON。
- 扫描未发现本地机器路径、内部域名、带凭据 URL 或常见密钥形态。
- README 首页不再放“出师证书”；鲁班打磨结果放在 [docs/luban-showcase.md](docs/luban-showcase.md)。
