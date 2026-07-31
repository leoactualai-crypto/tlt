# Use Async Message Send for Real-Time Discord Communication: Message Delivery Functions

These rules are ALWAYS ACTIVE for all Discord adapter components that send messages to channels, threads, or users, including reminder.py and bot_manager.py.

### Rules

- **R-MSG-001** MUST: Message delivery functions MUST be declared as async def to support non-blocking I/O.
- **R-MSG-002** MUST: All .send() calls in Discord adapter files MUST be preceded by the await keyword.
- **R-MSG-003** MUST: HTTP client operations to TLT service endpoints MUST use aiohttp.AsyncClient with async context managers (async with) for proper resource cleanup.
- **R-MSG-004** MUST: Discord event handlers (on_ready, on_guild_join, on_reaction_add) MUST be declared as async def.
- **R-MSG-005** MUST: Reminder scheduling and delivery functions MUST be declared as async def.
- **R-MSG-006** SHOULD: Use asyncio.gather() to parallelize independent async operations such as sending multiple messages or querying multiple endpoints.
- **R-MSG-007** SHOULD: Implement proper exception handling within async functions using try/except blocks and log errors using the logging module (logging.getLogger(__name__)).
- **R-MSG-008** MAY: Synchronous utility functions that do not perform I/O may remain synchronous.

### Verify

```bash
# Count await calls with .send() operations
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Count .send() calls without await (should be zero in async contexts)
grep -r '\.send(' monorepo/tlt/adapters/discord_adapter/ | grep -v 'await' | grep -v '#' | wc -l

# Run async integration tests
python -m pytest monorepo/tlt/adapters/discord_adapter/tests/ -k 'async' -v

# Check for blocking operations in async functions
python -m pylint --disable=all --enable=not-async-context-manager,await-outside-async monorepo/tlt/adapters/discord_adapter/
```

**Accept when:**
- All .send() calls in Discord adapter files are preceded by await keyword
- No synchronous blocking calls to Discord API are present in async functions
- Async integration tests pass without event loop blocking warnings
- Static analysis using pylint async rules reports no violations
- Event loop responsiveness is maintained during high-volume message delivery operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All message delivery functions must be audited for async/await compliance before merge. Code review MUST block merge if blocking I/O is found in async functions without documented exception approval.
</enforcement>