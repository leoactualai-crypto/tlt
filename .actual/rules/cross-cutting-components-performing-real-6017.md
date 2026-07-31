# Use Standard Logging with Named Loggers for Real-Time Discord Operations: Components Performing Real

These rules are ALWAYS ACTIVE for all Discord adapter components that perform real-time message operations and require operational observability.

### Rules

- **R-DISCORD-LOG-001** MUST: Components performing real-time operations (await thread.send, channel.send, user.send) MUST log operation initiation and completion status.

### Verify

```bash
# Verify standard logging pattern adoption
grep -r 'logging.getLogger(__name__)' monorepo/tlt/adapters/discord_adapter/

# Count logging imports in Discord adapter
grep -r 'import logging' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Run Discord adapter tests with log output
python -m pytest tests/ -k discord_adapter -v --log-cli-level=INFO
```

**Accept when:**
- All Discord adapter modules (reminder.py, bot_manager.py) contain `logging.getLogger(__name__)` initialization at module level
- Real-time operations (await send, HTTP client calls) have corresponding log statements at INFO or DEBUG level with context IDs (reminder_id, message_id, guild_id)
- Tests pass with log output visible and no custom logging framework imports detected in Discord adapter modules
- Log messages include relevant context for traceability across async execution contexts

<enforcement>
Claude Code MUST NOT skip or defer verification. All Discord adapter modules must be inspected for logger initialization and operation logging before approval.
</enforcement>