# Architecture - Claude Code Telegram

## Purpose

Telegram bot providing remote access to Claude Code. Users chat naturally with Claude via Telegram (agentic mode) or use a classic terminal-mode interface with 13 commands and inline keyboards.

## Tech Stack

- **Language**: Python 3.10+, managed with Poetry
- **Telegram**: `python-telegram-bot`
- **Claude**: `claude-agent-sdk` with `ClaudeSDKClient` for async streaming
- **Storage**: SQLite via aiosqlite (repository pattern)
- **Logging**: structlog (JSON in prod, console in dev)
- **Config**: Pydantic Settings v2 with env detection and feature flags
- **Webhooks**: FastAPI server for GitHub HMAC-SHA256 + Bearer token auth
- **Scheduler**: APScheduler with SQLite persistence

## Project Structure

```
src/
  config/          # Pydantic Settings, feature flags, YAML project loader
  bot/
    handlers/      # Telegram command, message, callback handlers (classic mode)
    middleware/     # Auth, rate limit, security input validation
    features/      # Git integration, file handling, quick actions, session export
    orchestrator.py  # Routes to agentic or classic handlers, project-topic routing
  claude/
    facade.py      # ClaudeIntegration (high-level facade)
    sdk_integration.py  # ClaudeSDKManager wrapping claude-agent-sdk
    monitor.py     # ToolMonitor - validates tool calls against allowlists
  projects/        # Multi-project support: YAML registry, thread manager
  storage/         # SQLite repos (users, sessions, messages, tool_usage, audit_log, cost_tracking)
  security/        # Auth (whitelist + token), input validators, rate limiter, audit logging
  events/          # EventBus (async pub/sub), event types, AgentHandler
  api/             # FastAPI webhook server
  scheduler/       # APScheduler cron jobs
  notifications/   # NotificationService, rate-limited Telegram delivery
```

## Request Flow

**Agentic mode** (default, `AGENTIC_MODE=true`):

```
Telegram message -> Security middleware (group -3) -> Auth middleware (group -2)
-> Rate limit (group -1) -> MessageOrchestrator.agentic_text() (group 10)
-> ClaudeIntegration.run_command() -> SDK
-> Response parsed -> Stored in SQLite -> Sent back to Telegram
```

**External triggers** (webhooks, scheduler):

```
Webhook POST /webhooks/{provider} -> Signature verification -> Deduplication
-> Publish WebhookEvent to EventBus -> AgentHandler.handle_webhook()
-> ClaudeIntegration.run_command() -> Publish AgentResponseEvent
-> NotificationService -> Rate-limited Telegram delivery
```

**Classic mode** (`AGENTIC_MODE=false`): Same middleware chain, routes through full command/message handlers in `src/bot/handlers/` with 13 commands and inline keyboards.

## Key Patterns

### Claude SDK Integration

`ClaudeIntegration` (facade) wraps `ClaudeSDKManager`, which uses `claude-agent-sdk` with `ClaudeSDKClient` for async streaming. Session IDs come from Claude's `ResultMessage`, not generated locally. Sessions auto-resume per user+directory, persisted in SQLite.

### Dependency Injection

Bot handlers access dependencies via `context.bot_data`:
```python
context.bot_data["auth_manager"]
context.bot_data["claude_integration"]
context.bot_data["storage"]
context.bot_data["security_validator"]
```

### Security Model

5-layer defense:
1. **Authentication** - whitelist/token
2. **Directory isolation** - `APPROVED_DIRECTORY` + path traversal prevention
3. **Input validation** - blocks `..`, `;`, `&&`, `$()`, etc. (relaxable with `DISABLE_SECURITY_PATTERNS=true`)
4. **Rate limiting** - token bucket
5. **Audit logging**

`ToolMonitor` validates Claude's tool calls against allowlist/disallowlist, file path boundaries, and dangerous bash patterns. Bypassed with `DISABLE_TOOL_VALIDATION=true`.

Webhook auth: GitHub HMAC-SHA256 signature verification, generic Bearer token for other providers, atomic deduplication via `webhook_events` table.

### DateTime Convention

All datetimes use timezone-aware UTC: `datetime.now(UTC)` (not `datetime.utcnow()`). SQLite adapters auto-convert TIMESTAMP/DATETIME columns. Model `from_row()` methods must guard `fromisoformat()` with `isinstance(val, str)` checks.

### Multi-Project Topics

When `ENABLE_PROJECT_THREADS=true`, messages route to per-project Telegram forum topics. Config in `projects.yaml`. `/sync_threads` reconciles topics.
