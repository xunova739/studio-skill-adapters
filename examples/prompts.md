# Example Prompts

## Codex

```text
studio，帮我看这个项目现在处于哪个阶段，下一步该做什么
```

Expected behavior:

- read `planning/status.json` first, then `.planning/status.json`;
- report current stage, evidence, and the smallest useful next action;
- avoid Claude hooks or background cron assumptions.

```text
开启本地 Studio，但不要装 Claude hooks
```

Expected behavior:

- write or propose only the `STUDIO:LOCAL-CODEX` marker block in `AGENTS.md`;
- do not write `~/.claude`;
- do not push or deploy unless the project policy asks for it.

## Claude Code

```text
studio，检查这个 Claude Code 项目当前阶段
```

Expected behavior:

- read `.planning/status.json` first, then `planning/status.json`;
- report current stage and evidence;
- use `CLAUDE.md` as the local routing file.

```text
旧 skill 的 target_repo 换成我们上传 GitHub 的仓库
```

Expected behavior:

- use a credential-free GitHub mirror URL;
- never place tokens, cookies, passwords, or private keys in skill files;
- treat GitHub as source reference when the runtime environment is offline.
