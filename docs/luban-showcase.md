# Luban Showcase Notes

## 1. Material Check

Challenge 1 - Real problem: valid. Teams often need one internal workflow to work across multiple agent runtimes without copying unsafe runtime state.

Challenge 2 - Unique angle: the value is not a brand-new automation engine. The value is a safe adapter split: Codex gets `AGENTS.md`; Claude Code gets `CLAUDE.md`; old hooks and sync URLs are not copied blindly.

Challenge 3 - Installation reason: users install this when they already have a Studio workflow and need a clean, shareable bridge.

Challenge 4 - Public showcase: the visible artifacts are two skills, eval prompts, example state, security contract, and a release card.

Conclusion: good material for a small public utility repository, not a giant framework.

## 2. Quick Ecosystem Scan

This release pass used a light ecosystem scan to shape public positioning:

| Reference | Link | What It Tells Us |
|---|---|---|
| Agent skill security research | https://arxiv.org/abs/2605.11418 | `SKILL.md` text is operational metadata, so descriptions and safety boundaries matter. |
| Agent skill ecosystem analysis | https://arxiv.org/abs/2602.08004 | Skills are becoming a reusable infrastructure layer, but many are redundant or shallow. |
| Agentic coding configuration study | https://arxiv.org/abs/2602.14690 | Context files such as `AGENTS.md` are common cross-tool configuration surfaces. |
| OpenClaw skills guide | https://www.techradar.com/pro/what-are-openclaw-skills-a-detailed-guide | Public skill users expect clear install scope, local execution boundaries, and security caution. |
| GitHub agent integration coverage | https://www.theverge.com/news/873665/github-claude-codex-ai-agents | Cross-agent workflows are becoming normal, so runtime-specific adapters are easier to explain than one mixed bundle. |

## 3. Positioning

Old positioning:

```text
Autonomous Studio bundle with hooks, state, sync, and runtime assumptions.
```

New positioning:

```text
Two safe local adapters for carrying one Studio workflow across Codex and Claude Code.
```

The repository should compete on safety, clarity, and installation confidence, not on promising full autonomous delivery.

## 4. Quality Score

| Dimension | Weight | Before | After | Evidence |
|---|---:|---:|---:|---|
| Frontmatter and trigger clarity | 7 | 4 | 6 | Runtime-specific descriptions now state when to use each adapter. |
| Workflow clarity | 12 | 7 | 9 | Both adapters expose stage routing and status-file rules. |
| Failure-mode encoding | 12 | 4 | 8 | Hooks, credentials, GitHub offline mode, and human override are explicit. |
| Checkpoints | 6 | 3 | 5 | Status example and marker blocks define review points. |
| Executability | 17 | 9 | 12 | Quick start, file structure, and eval prompts are included. |
| Resource integration | 4 | 1 | 3 | Examples and docs are now present. |
| Architecture | 12 | 6 | 10 | Runtime split removes mixed Codex/Claude assumptions. |
| Tested behavior | 23 | 8 | 14 | `jq` and secret scans are recorded; live runtime reload remains untested. |
| Anti-pattern coverage | 7 | 3 | 6 | Security file and README list what the adapters do not do. |
| Total | 100 | 45 | 73 | Estimated after release polish; not a full live runtime score. |

## 5. Release Gaps

P0:

- Replace `https://github.com/<org>/<repo>.git` with the real mirror repository when known.
- Keep the repository private until the owner decides whether internal workflow names are public-safe.

P1:

- Add a short terminal recording or GIF showing copy-install-trigger.
- Add one real before/after example from a safe demo project.

P2:

- Add a release tag after the first user confirms both adapters load in their target runtimes.

## 6. Release Card

```text
┌─────────────────────────────────────┐
│  出师证书 · 鲁班工坊                │
│                                     │
│  作品：Studio Skill Adapters        │
│  过尺：45 分 → 73 分（预估）        │
│  定位：一套 Studio，两种安全适配    │
│  绝活：Codex/Claude 分面，不带凭据  │
│  下一步：替换真实 GitHub mirror     │
│                                     │
│  验收师傅：鲁班                     │
└─────────────────────────────────────┘
```
