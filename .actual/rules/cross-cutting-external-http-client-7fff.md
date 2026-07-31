# Use Standard Logging with Named Loggers for Real-Time Discord Operations: External Http Client

These rules are ALWAYS ACTIVE for all Discord adapter components that perform real-time message operations and require operational observability, including reminder.py, bot_manager.py, and related modules handling asynchronous Discord operations and external HTTP client interactions.

### Rules

- **R-DISCORD-LOG-001** MUST: External HTTP client calls to service endpoints MUST log request initiation, response status, and error conditions.
- **R-DISCORD-LOG-002** MUST: Initialize module logger at top of each Discord adapter file using `logger = logging.getLogger(__name__)`.
- **R-DISCORD-LOG-003** MUST: Log real-time operation boundaries with `logger.info()` before await send operations and `logger.error()` for exceptions with context IDs.
- **R-DISCORD-LOG-004** SHOULD: Include relevant context in log messages: reminder_id for reminder operations, message_id/channel_id for Discord operations, guild_id for bot events.
- **R-DISCORD-LOG-005** SHOULD: Use `logger.exception()` in exception handlers to automatically capture stack traces for debugging.
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
- External HTTP client calls include logging for request initiation, response status, and error conditions

<enforcement>
Clause Code MUST NOT skip or defer verification. All rules in this file are mandatory for Discord adapter code paths.
</enforcement>