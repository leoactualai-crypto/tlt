# Use Async Message Send for Real-Time Discord Communication: Discord Adapter Components

These rules are ALWAYS ACTIVE for all Discord adapter components that send messages to channels, threads, or users, including reminder.py and bot_manager.py.

### Rules

- **R-DCC5-001** MUST NOT: Discord adapter components MUST NOT use synchronous blocking calls for message delivery.
- **R-DCC5-002** MUST: All new Discord message send operations must use await with channel.send(), thread.send(), or user.send() methods.
- **R-DCC5-003** MUST: HTTP client operations should use aiohttp.AsyncClient with async context managers (async with) for proper resource cleanup.
- **R-DCC5-004** SHOULD: Use asyncio.gather() to parallelize independent async operations such as sending multiple messages or querying multiple endpoints.
- **R-DCC5-005** MUST: Implement proper exception handling within async functions using try/except blocks and log errors using the logging module (logging.getLogger(__name__)).

### Verify

```bash
# Count await calls with .send() in Discord adapter
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Count .send() calls without await (excluding comments)
grep -r '\.send(' monorepo/tlt/adapters/discord_adapter/ | grep -v 'await' | grep -v '#' | wc -l

# Run async integration tests
python -m pytest monorepo/tlt/adapters/discord_adapter/tests/ -k 'async' -v
```

**Accept when:**
- All .send() calls in Discord adapter files are preceded by await keyword
- No synchronous blocking calls to Discord API are present in async functions
- Async integration tests pass without event loop blocking warnings
- Static analysis using pylint async rules detects no violations (not-async-context-manager, await-outside-async)

<enforcement>
Claude Code MUST NOT skip or defer verification. All Discord adapter message send operations MUST comply with async/await patterns. Violations block merge and trigger CI pipeline failures.
</enforcement>