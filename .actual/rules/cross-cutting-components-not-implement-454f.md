# Use Standard Logging with Named Loggers for Real-Time Discord Operations: Components Not Implement

These rules are ALWAYS ACTIVE for all Discord adapter components that perform real-time message operations and require operational observability, including reminder.py, bot_manager.py, and related modules handling asynchronous Discord operations, HTTP client interactions, and scheduled tasks.

### Rules

- **R-LOGGING-001** MUST_NOT: Components MUST NOT implement custom logging frameworks or wrappers that bypass Python's standard logging module.
- **R-LOGGING-002** MUST: Initialize module logger at top of each Discord adapter file using `logger = logging.getLogger(__name__)`.
- **R-LOGGING-003** MUST: Log real-time operation boundaries with `logger.info()` before await send operations and `logger.error()` for exceptions with context IDs.
- **R-LOGGING-004** MUST: Include relevant context in log messages: reminder_id for reminder operations, message_id/channel_id for Discord operations, guild_id for bot events.
- **R-LOGGING-005** SHOULD: Configure logging level via environment variable or config file to allow runtime adjustment without code changes.
- **R-LOGGING-006** SHOULD: Use `logger.exception()` in exception handlers to automatically capture stack traces for debugging.

### Verify

```bash
# Verify standard logging pattern adoption
grep -r 'logging.getLogger(__name__)' monorepo/tlt/adapters/discord_adapter/

# Count logging imports
grep -r 'import logging' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Run Discord adapter tests with log visibility
python -m pytest tests/ -k discord_adapter -v --log-cli-level=INFO

# Verify no custom logging framework imports in Discord adapter
grep -r 'from.*logging' monorepo/tlt/adapters/discord_adapter/ | grep -v 'import logging' | grep -v 'getLogger'
```

**Accept when:**
- All Discord adapter modules contain `logging.getLogger(__name__)` initialization at module level
- Real-time operations (await send, HTTP client calls) have corresponding log statements at INFO or DEBUG level
- Tests pass with log output visible and no custom logging framework imports detected in Discord adapter modules
- No custom logging wrappers or abstractions are present in Discord adapter code
- Context IDs (reminder_id, message_id, guild_id) are included in log messages for real-time operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for Discord adapter components. Code review MUST verify logger initialization and operation logging before merge approval. CI pipeline MUST warn on missing logger initialization in Discord adapter modules.
</enforcement>