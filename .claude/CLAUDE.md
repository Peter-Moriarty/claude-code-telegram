# Claude Code Telegram

Telegram bot for remote Claude Code access. Python 3.10+, Poetry, python-telegram-bot, claude-agent-sdk.

## Docs

| Doc | Contents |
|---|---|
| `docs/architecture.md` | Purpose, tech stack, project structure, request flow, key patterns, security model |
| `docs/deployment.md` | GCE instance, GSM config, systemd services, env vars, rate limiting |
| `docs/gotchas.md` | Known quirks, safety rules, datetime convention, code style |
| `docs/configuration.md` | Full configuration guide with all env vars and feature flags |
| `docs/setup.md` | Installation, auth setup, agentic platform setup |
| `docs/tools.md` | Allowed tools reference and security layers |

## Commands

```bash
make dev          # Install all deps (including dev)
make run          # Run the bot
make test         # Run tests with coverage
make lint         # Black + isort + flake8 + mypy
make format       # Auto-format with black + isort
```
