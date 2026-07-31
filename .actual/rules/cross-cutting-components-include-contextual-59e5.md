# Use Standard Logging with Named Loggers for Real-Time Discord Operations: Components Include Contextual

These rules are ALWAYS ACTIVE for all Discord adapter components that perform real-time message operations and require operational observability, including reminder.py, bot_manager.py, and related async Discord integration modules.

### Rules

- **R-DISCORD-LOG-001** MUST: Initialize module-level named loggers using `logger = logging.getLogger(__name__)` at the top of each Discord adapter file.
- **R-DISCORD-LOG-002** MUST: Log real-time operation boundaries with logger.info() before await send operations and logger.error() for exceptions with context IDs.
- **R-DISCORD-LOG-003** MAY: Components MAY include contextual metadata (reminder_id, message_id, guild_id, user_id) in log messages to aid correlation.
- **R-DISCORD-LOG-004** SHOULD: Use logger.exception() in exception handlers to automatically capture stack traces for debugging.
- **R-DISCORD-LOG-005** SHOULD: Configure logging level via environment variable or config file to allow runtime adjustment without code changes.

### Verify

```bash
# Verify all Discord adapter modules contain logging.getLogger(__name__) initialization
grep -r 'logging.getLogger(__name__)' monorepo/tlt/adapters/discord_adapter/

# Count logging imports in Discord adapter modules
grep -r 'import logging' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Run Discord adapter tests with log output visible
python -m pytest tests/ -k discord_adapter -v --log-cli-level=INFO
```

**Accept when:**
- All Discord adapter modules contain `logging.getLogger(__name__)` initialization
- Real-time operations (await send, HTTP client calls) have corresponding log statements at INFO or DEBUG level
- Tests pass with log output visible and no custom logging framework imports detected in Discord adapter modules
- Context IDs (reminder_id, message_id, guild_id, user_id) are included in log messages for real-time operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All Discord adapter modules MUST be checked for proper logger initialization and operation logging before code review approval.
</enforcement>