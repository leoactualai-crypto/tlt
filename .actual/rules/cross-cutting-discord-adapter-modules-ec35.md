# Use Standard Logging with Named Loggers for Real-Time Discord Operations: Discord Adapter Modules

These rules are ALWAYS ACTIVE for all Discord adapter modules that perform real-time message operations and require operational observability, including reminder.py, bot_manager.py, and related async Discord integration components.

### Rules

- **R-DISCORD-LOG-001** MUST: All Discord adapter modules MUST obtain loggers using `logging.getLogger(__name__)` to ensure module-scoped logger naming.
- **R-DISCORD-LOG-002** MUST: Real-time operation boundaries (await send operations, HTTP client calls) MUST have corresponding log statements at INFO or DEBUG level.
- **R-DISCORD-LOG-003** MUST: Log messages MUST include relevant context identifiers such as reminder_id, message_id, channel_id, or guild_id for traceability.
- **R-DISCORD-LOG-004** SHOULD: Use `logger.exception()` in exception handlers to automatically capture stack traces for debugging.
- **R-DISCORD-LOG-005** SHOULD: Configure logging level via environment variable or config file to allow runtime adjustment without code changes.

### Verify

```bash
# Verify all Discord adapter modules contain logger initialization
grep -r 'logging.getLogger(__name__)' monorepo/tlt/adapters/discord_adapter/

# Count logging imports in Discord adapter modules
grep -r 'import logging' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Run Discord adapter tests with log output visible
python -m pytest tests/ -k discord_adapter -v --log-cli-level=INFO

# Verify no custom logging framework imports in Discord adapter modules
grep -r 'from structlog\|from loguru\|import structlog\|import loguru' monorepo/tlt/adapters/discord_adapter/ || echo "No custom logging frameworks detected"
```

**Accept when:**
- All Discord adapter modules contain `logging.getLogger(__name__)` initialization at module level
- Real-time operations (await send, HTTP client calls) have corresponding log statements at INFO or DEBUG level
- Tests pass with log output visible and no custom logging framework imports detected in Discord adapter modules
- Log messages include context identifiers (reminder_id, message_id, guild_id) for traceability
- No print statements or Discord channel logging used for operational visibility in production code

<enforcement>
Claude Code MUST NOT skip or defer verification. All Discord adapter modules MUST be checked for logger initialization and operation logging before approval. Violations require code review feedback requesting addition of standard logging before merge approval.
</enforcement>