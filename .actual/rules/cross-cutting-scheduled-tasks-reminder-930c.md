# Use Standard Logging with Named Loggers for Real-Time Discord Operations: Scheduled Tasks Reminder

These rules are ALWAYS ACTIVE for all Discord adapter components that perform real-time message operations and require operational observability, including reminder.py, bot_manager.py, and related async Discord integration modules.

### Rules

- **R-DISCORD-LOG-001** SHOULD: Scheduled tasks (reminder_check_task) SHOULD log task start, iteration count, and completion or failure states.
- **R-DISCORD-LOG-002** MUST: Initialize module logger at top of each Discord adapter file using `logger = logging.getLogger(__name__)`.
- **R-DISCORD-LOG-003** SHOULD: Log real-time operation boundaries with `logger.info()` before await send operations and `logger.error()` for exceptions with context IDs.
- **R-DISCORD-LOG-004** SHOULD: Include relevant context in log messages: reminder_id for reminder operations, message_id/channel_id for Discord operations, guild_id for bot events.
- **R-DISCORD-LOG-005** SHOULD: Use `logger.exception()` in exception handlers to automatically capture stack traces for debugging.
- **R-DISCORD-LOG-006** MAY: Configure logging level via environment variable or config file to allow runtime adjustment without code changes.

### Verify

```bash
# Verify all Discord adapter modules contain logging.getLogger(__name__) initialization
grep -r 'logging.getLogger(__name__)' monorepo/tlt/adapters/discord_adapter/

# Count logging imports in Discord adapter modules
grep -r 'import logging' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Run Discord adapter tests with INFO-level logging
python -m pytest tests/ -k discord_adapter -v --log-cli-level=INFO
```

**Accept when:**
- All Discord adapter modules contain `logging.getLogger(__name__)` initialization at module level
- Real-time operations (await send, HTTP client calls, scheduled tasks) have corresponding log statements at INFO or DEBUG level
- Tests pass with log output visible and no custom logging framework imports detected in Discord adapter modules
- Scheduled task logging includes task start, iteration count, and completion/failure states
- Exception handlers use `logger.exception()` to capture stack traces

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review checklist MUST verify logger initialization and operation logging. Automated CI checks MUST detect missing `logging.getLogger(__name__)` pattern in new Discord adapter modules. Integration tests MUST execute with log level validation before merge approval.
</enforcement>