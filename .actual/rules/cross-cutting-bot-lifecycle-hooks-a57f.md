# Use Standard Logging with Named Loggers for Real-Time Discord Operations: Bot Lifecycle Hooks

These rules are ALWAYS ACTIVE for all Discord adapter components that perform real-time message operations and require operational observability, including reminder.py, bot_manager.py, and related modules handling asynchronous Discord operations, bot lifecycle hooks, and scheduled tasks.

### Rules

- **R-DISCORD-LOG-001** SHOULD: Bot lifecycle hooks (on_ready, on_guild_join, on_guild_remove, on_reaction_add) SHOULD log event receipt and processing outcomes.
- **R-DISCORD-LOG-002** MUST: Initialize module logger at top of each Discord adapter file using `logger = logging.getLogger(__name__)`.
- **R-DISCORD-LOG-003** SHOULD: Log real-time operation boundaries with logger.info before await send operations and logger.error for exceptions with context IDs.
- **R-DISCORD-LOG-004** SHOULD: Include relevant context in log messages: reminder_id for reminder operations, message_id/channel_id for Discord operations, guild_id for bot events.
- **R-DISCORD-LOG-005** SHOULD: Use logger.exception() in exception handlers to automatically capture stack traces for debugging.
- **R-DISCORD-LOG-006** MAY: Configure logging level via environment variable or config file to allow runtime adjustment without code changes.

### Verify

```bash
# Verify logger initialization in Discord adapter modules
grep -r 'logging.getLogger(__name__)' monorepo/tlt/adapters/discord_adapter/

# Count logging imports
grep -r 'import logging' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Run Discord adapter tests with log output
python -m pytest tests/ -k discord_adapter -v --log-cli-level=INFO
```

**Accept when:**
- All Discord adapter modules contain `logging.getLogger(__name__)` initialization
- Real-time operations (await send, HTTP client calls) have corresponding log statements at INFO or DEBUG level
- Tests pass with log output visible and no custom logging framework imports detected in Discord adapter modules
- No print statements or Discord channel logging used for operational visibility in production code

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review checklist MUST verify logger initialization and operation logging. Automated CI checks MUST detect logging.getLogger(__name__) pattern in new Discord adapter modules. Integration tests MUST execute with log level validation before merge approval.
</enforcement>