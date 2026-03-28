# Deployment - Claude Code Telegram

## GCE Instance

- **Instance**: `agent-peter-claude-code` (`agent-peter-888888`, `europe-southwest1-a`)
- **SA**: `claude-code-vm@`
- **SSH**: `gcloud compute ssh peter@agent-peter-claude-code --zone=europe-southwest1-a --project=agent-peter-888888 --tunnel-through-iap`
- **Bot repo on VM**: `/home/peter/claude-code-telegram/`
- **Child projects on VM**: `/home/peter/projects/agent-peter/`

## Configuration

Fully GSM-based. Secret `claude-telegram-bot-env` (27 vars) fetched at service start. No `.env` file on the VM.

- **Update config**: `gcloud secrets versions add claude-telegram-bot-env --project=agent-peter-888888 --data-file=-`, then `sudo systemctl restart claude-telegram-bot`
- **Fetch script**: `/usr/local/bin/fetch-bot-env.sh` - pulls GSM secret to tmpfs `/run/claude-bot/env`

## systemd Services

| Service | Purpose |
|---|---|
| `claude-telegram-bot` | Bot daemon |
| `claude-remote-control` | Remote-control watchdog |
| `claude-code-update` | Daily CLI update (04:00 UTC) |

Sudoers configured at `/etc/sudoers.d/claude-remote-control` - peter can restart/start/stop services without password.

## Environment Variables

### Required

| Variable | Description |
|---|---|
| `TELEGRAM_BOT_TOKEN` | Bot token from @BotFather |
| `TELEGRAM_BOT_USERNAME` | Bot username (e.g. `overlord_cc_bot`) |
| `APPROVED_DIRECTORY` | Root directory for file access |

### Key Optional

| Variable | Default | Description |
|---|---|---|
| `ALLOWED_USERS` | (none) | Comma-separated Telegram user IDs |
| `ANTHROPIC_API_KEY` | (none) | API key (uses CLI auth if omitted) |
| `AGENTIC_MODE` | `true` | Conversational mode vs classic terminal mode |
| `ENABLE_MCP` | `false` | Model Context Protocol support |
| `MCP_CONFIG_PATH` | (none) | Path to MCP config JSON |
| `ENABLE_API_SERVER` | `false` | FastAPI webhook server |
| `API_SERVER_PORT` | `8080` | Webhook server port |
| `GITHUB_WEBHOOK_SECRET` | (none) | GitHub HMAC-SHA256 secret |
| `WEBHOOK_API_SECRET` | (none) | Bearer token for generic providers |
| `ENABLE_SCHEDULER` | `false` | Cron job scheduler |
| `NOTIFICATION_CHAT_IDS` | (none) | Telegram chat IDs for proactive notifications |
| `DISABLE_SECURITY_PATTERNS` | `false` | Relax input validation (trusted envs only) |
| `DISABLE_TOOL_VALIDATION` | `false` | Skip tool allowlist checks |
| `VERBOSE_LEVEL` | `1` | Output verbosity (0-2) |

### Project Thread Mode

| Variable | Default | Description |
|---|---|---|
| `ENABLE_PROJECT_THREADS` | `false` | Strict project routing via topics |
| `PROJECT_THREADS_MODE` | `private` | `private` or `group` |
| `PROJECT_THREADS_CHAT_ID` | (none) | Required for group mode |
| `PROJECTS_CONFIG_PATH` | (none) | Path to YAML project registry |
| `PROJECT_THREADS_SYNC_ACTION_INTERVAL_SECONDS` | `1.1` | Delay between topic sync API calls |

### Rate Limiting

| Variable | Default | Description |
|---|---|---|
| `RATE_LIMIT_REQUESTS` | `10` | Requests per window |
| `RATE_LIMIT_WINDOW` | `60` | Window in seconds |
| `RATE_LIMIT_BURST` | `20` | Burst capacity |
| `CLAUDE_MAX_COST_PER_USER` | `10.0` | Max cost per user (USD) |

### Environment Overrides

- **Development**: `DEBUG=true` - sets `LOG_LEVEL=DEBUG`, `rate_limit_requests=100`, `claude_timeout_seconds=600`
- **Testing**: `ENVIRONMENT=testing` - in-memory SQLite, `/tmp/test_projects`, `rate_limit_requests=1000`
- **Production**: `ENVIRONMENT=production` - `rate_limit_requests=5`, `claude_max_cost_per_user=5.0`, `session_timeout_hours=12`

## Telegram Group

- **Group**: "Overlord CC" (chat ID `-1003852608812`), 17 forum topics, strict routing
- **Config**: `projects.yaml` in repo root
- **Sync**: `/sync_threads` to reconcile
