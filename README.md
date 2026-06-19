# Studio Skill Adapters

> Turn one internal Studio workflow into two clean, reviewable local skills: one for Codex, one for Claude Code.

![Runtime](https://img.shields.io/badge/runtime-Codex%20%7C%20Claude%20Code-blue)
![Scope](https://img.shields.io/badge/scope-project--local-brightgreen)
![Secrets](https://img.shields.io/badge/secrets-not%20embedded-critical)

## When You Need This

- You have a mature Studio workflow, but the original bundle mixes runtime state, hooks, sync URLs, and platform assumptions.
- You want one workflow that can run in Codex through `AGENTS.md`, and in Claude Code through `CLAUDE.md`.
- You need to publish or share the workflow without leaking internal repositories, local paths, tokens, or old session state.

## What This Repository Ships

| Adapter | Runtime | Entry File | What It Does |
|---|---|---|---|
| Codex Studio Adapter | Codex / Codex Desktop | `.codex/skills/autonomous-studio-codex/SKILL.md` | Routes discovery, PRD, planning, implementation, verification, review, and deploy handoff through Codex-safe project files. |
| Claude Studio Adapter | Claude Code | `.claude/skills/autonomous-studio-claude/SKILL.md` | Keeps the same Studio workflow but targets `CLAUDE.md` and treats hooks as optional, reviewed additions. |
| Showcase Notes | GitHub readers | `docs/luban-showcase.md` | Explains positioning, safety boundaries, and the release card used to judge publish readiness. |
| Workflow Map | Teams adopting Studio | `docs/workflow.md` | Shows the intended demand-to-agent-development flow. |
| Examples | New adopters | `examples/` | Provides sample status state and trigger prompts. |

## Quick Start

Clone the repository, then copy only the adapter you need into your target project.

```bash
git clone https://github.com/xunova739/studio-skill-adapters.git
cd studio-skill-adapters

# Codex project
mkdir -p /path/to/project/.codex/skills
cp -R .codex/skills/autonomous-studio-codex /path/to/project/.codex/skills/

# Claude Code project
mkdir -p /path/to/project/.claude/skills
cp -R .claude/skills/autonomous-studio-claude /path/to/project/.claude/skills/
```

Then restart or reload the target agent environment if its skill list is cached.

## Trigger Examples

Use natural language like:

- `studio，帮我判断这个项目现在处于哪个阶段`
- `开启本地 Studio，但不要装 Claude hooks`
- `把旧 skill 里的内部仓库换成我们上传 GitHub 的仓库`
- `从需求到上线，按 Studio 流程帮我推进`
- `PRD 写完了，下一步进入技术方案和验证计划`
- `检查一下这个 skill 有没有把 token 或内部 URL 带出去`

## Visible Output

The adapters are designed to produce visible, reviewable artifacts instead of hidden automation:

```text
User request
  -> read planning/status.json or .planning/status.json
  -> identify current stage and evidence
  -> select the next workflow lane
  -> write only a short AGENTS.md or CLAUDE.md marker block when requested
  -> preserve human approval points for high-impact decisions
```

For AI-assisted recruiting, screening, ranking, or candidate analysis workflows, the default posture is:

```text
AI assists. Humans decide.
```

Every AI recommendation should expose evidence, score/reason, and a human override path.

## End-to-End Flow

The intended Studio flow is:

```text
Talk through the demand
  -> write PRD.html
  -> human confirms the PRD.html
  -> write PRD.json only after confirmation
  -> split technical plan and acceptance checks
  -> develop through the appropriate agent skill
  -> verify, review, and fix
  -> deploy or hand off
  -> feed decisions and lessons back into the next cycle
```

See [docs/workflow.md](docs/workflow.md) for the detailed stage map.

## GitHub Mirror Configuration

Replace any old internal `target_repo` or sync block with a credential-free GitHub mirror reference:

```yaml
studio_source:
  kind: github-mirror
  repo: "https://github.com/<org>/<repo>.git"
  branch: "main"
  credential_rule: "Do not embed credentials in skill files."
```

Authentication belongs in GitHub CLI, SSH keys, or the system credential helper. Do not store tokens, cookies, private keys, passwords, or auth tickets in skill files.

## Why This Is Different

| Common Problem | This Repository's Choice |
|---|---|
| One giant bundle is copied into every runtime. | Two thin adapters keep Codex and Claude Code concerns separate. |
| Hooks run before anyone understands them. | Hooks are optional and require explicit review in the Claude adapter. |
| Internal sync URLs leak into public docs. | GitHub mirror configuration is credential-free and placeholder-based. |
| Root context files become long manuals. | `AGENTS.md` and `CLAUDE.md` receive only short marker-bounded routing blocks. |
| AI automation becomes a black box. | Every high-impact AI decision needs evidence, reason, and human override. |

## Safety Boundaries

These adapters do not:

- install Claude hooks automatically;
- copy old sessions, decision logs, checkpoints, or memory dumps;
- embed credentials in repository URLs;
- push code or deploy without the current project policy or explicit user instruction;
- make black-box decisions for candidate screening or other high-impact workflows.

See [SECURITY.md](SECURITY.md) for the public safety contract.

## File Structure

```text
.
├── .codex/skills/autonomous-studio-codex/
│   ├── SKILL.md
│   └── evals/evals.json
├── .claude/skills/autonomous-studio-claude/
│   ├── SKILL.md
│   └── evals/evals.json
├── docs/luban-showcase.md
├── docs/workflow.md
├── examples/prompts.md
├── examples/status.example.json
├── ALIASES.md
├── SECURITY.md
└── README.md
```

## Validation

Current release checks:

- `jq` parsed both eval files.
- Secret scan found no internal repository domains, credential-shaped URLs, hardcoded key names, or common API-key patterns.
- The public README avoids local machine paths and internal source URLs.

## Release Card

```text
┌─────────────────────────────────────┐
│  Luban Release Card                 │
│                                     │
│  Work: Studio Skill Adapters        │
│  Position: two safe local adapters  │
│  Best move: split by runtime        │
│  Safety: no credentials embedded    │
│  Next: replace <org>/<repo> mirror  │
└─────────────────────────────────────┘
```
