# Gotchas - Claude Code Telegram

## Known Quirks

- `Topic_not_modified` warnings on startup are harmless (topics already open)
- `StartLimitIntervalSec` systemd warning (in wrong section) - cosmetic only

## Safety Rules

- Never commit secrets to version control. All secrets are in GSM.
- `DISABLE_SECURITY_PATTERNS` and `DISABLE_TOOL_VALIDATION` are for trusted environments only. Production should keep both `false` unless there's a specific reason.
- Webhook deduplication is atomic via `webhook_events` table - do not bypass.
- `SecurityValidator` blocks access to secrets (`.env`, `.ssh`, `id_rsa`, `.pem`) and dangerous shell patterns.

## DateTime

Always use `datetime.now(UTC)`, never `datetime.utcnow()` (deprecated). SQLite adapters auto-convert TIMESTAMP/DATETIME columns via `detect_types=PARSE_DECLTYPES`. Model `from_row()` methods must guard `fromisoformat()` calls with `isinstance(val, str)` checks.

## SDK Notes

- Session IDs come from Claude's `ResultMessage`, not generated locally
- Sessions auto-resume per user+directory, persisted in SQLite
- See `docs/SDK_DUPLICATION_REVIEW.md` for refactoring opportunities identified in the `src/claude/` module

## Bot Username

- `@overlord_cc_bot`

## Code Style

- Black (88 char line length), isort (black profile), flake8, mypy strict, autoflake for unused imports
- pytest-asyncio with `asyncio_mode = "auto"`
- structlog for all logging
- Type hints required on all functions (`disallow_untyped_defs = true`)
