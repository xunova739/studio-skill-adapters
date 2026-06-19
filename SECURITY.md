# Security Boundary

This repository is designed for project-local agent skills. Treat every copied skill as executable operational guidance.

## What Must Not Be Stored Here

- API keys, tokens, cookies, auth tickets, passwords, SSH private keys, or `.env` contents
- Credential-bearing Git URLs
- Private repository URLs that should not be public
- Local machine paths from a contributor's workstation
- Old agent sessions, decision logs, checkpoints, memory dumps, or runtime state

## Runtime Rules

- Codex adapter: do not install or execute Claude Code hooks.
- Claude Code adapter: hooks are optional and require explicit review before installation.
- Git push, deployment, webhook posting, and external API calls require the active project's normal approval policy.
- AI-assisted candidate screening or ranking must expose evidence, score/reason, and human override.

## Reporting Issues

If you find a leaked secret or unsafe default, rotate the secret outside this repository first, then open a private issue or contact the repository owner.
