# Example Prompts

## Codex

```text
studio，帮我看这个项目现在处于哪个阶段，下一步该做什么
```

Expected behavior:

- read `planning/status.json` first, then `.planning/status.json`;
- report current stage, evidence, and the smallest useful next action;
- avoid Claude hooks or background cron assumptions.
- separate state-machine progress from long-term memory.

```text
开启本地 Studio，但不要装 Claude hooks
```

Expected behavior:

- write or propose only the `STUDIO:LOCAL-CODEX` marker block in `AGENTS.md`;
- do not write `~/.claude`;
- do not push or deploy unless the project policy asks for it.

```text
单独生成 PRD.html，先不要写 PRD.json
```

Expected behavior:

- use the PM/spec flow to create or plan `PRD.html`;
- mark assumptions and open questions;
- stop before `PRD.json`;
- wait for explicit confirmation.

```text
PRD.html 大概可以，我们再聊一下
```

Expected behavior:

- stay in PRD review;
- do not create `PRD.json`;
- explain what would count as confirmation.

```text
这个是 Skill 调优，先写能暴露问题的测试场景再改
```

Expected behavior:

- use eval-first;
- define a realistic failure scenario and baseline;
- only then revise the skill text.

## Claude Code

```text
studio，检查这个 Claude Code 项目当前阶段
```

Expected behavior:

- read `.planning/status.json` first, then `planning/status.json`;
- report current stage and evidence;
- use `CLAUDE.md` as the local routing file.

```text
Check whether this skill is safe to publish.
```

Expected behavior:

- check that skill metadata does not contain credentials;
- check that no sessions, decision logs, checkpoints, or memory dumps are bundled;
- confirm that `PRD.html` and `PRD.json` are separate stages.

```text
安装 Studio hooks，但先告诉我会装什么、风险和怎么回滚
```

Expected behavior:

- list hook files, trigger timing, write scope, network behavior, and rollback plan;
- do not install until explicit approval;
- do not let hooks approve business intent, hiring decisions, publishing, or deployment.

```text
Run Studio verification headlessly and tell me if this is ready.
```

Expected behavior:

- run repeatable checks such as schema, evals, lint/typecheck/tests, Playwright, or sensitive scans;
- write evidence;
- say what still needs human approval.
