# Memory - Claude Code Telegram

Migrated from Overlord MEMORY.md (2026-03-05).

## GCE Instance

- **Instance**: `agent-peter-claude-code` (`agent-peter-888888`, `europe-southwest1-a`)
- **SA**: `claude-code-vm@`
- **SSH**: `gcloud compute ssh peter@agent-peter-claude-code --zone=europe-southwest1-a --project=agent-peter-888888 --tunnel-through-iap`

## Bot

- **Username**: `@overlord_cc_bot`
- **Repo**: `Peter-Moriarty/claude-code-telegram` at `/home/peter/claude-code-telegram/`
- **Projects**: All child repos cloned under `/home/peter/projects/agent-peter/`

## Configuration

- **Fully GSM-based**. Secret `claude-telegram-bot-env` (27 vars) fetched at service start. No `.env` file.
- **Update config**: `gcloud secrets versions add claude-telegram-bot-env --project=agent-peter-888888 --data-file=-`, then `sudo systemctl restart claude-telegram-bot`
- **Fetch script**: `/usr/local/bin/fetch-bot-env.sh` - pulls GSM secret to tmpfs `/run/claude-bot/env`

## systemd Services

- `claude-telegram-bot` (bot daemon)
- `claude-remote-control` (remote-control watchdog)
- `claude-code-update` (daily CLI update 04:00 UTC)

## Telegram Group

- **Group**: "Overlord CC" (chat ID `-1003852608812`), 17 forum topics, strict routing
- **Config**: `projects.yaml` in repo root
- **Sync**: `/sync_threads` to reconcile

## Sudoers

- `/etc/sudoers.d/claude-remote-control` - peter can restart/start/stop systemctl services without password

## Known Quirks

- `Topic_not_modified` warnings on startup are harmless (topics already open)
- `StartLimitIntervalSec` systemd warning (in wrong section) - cosmetic only
