# Use Async Message Send for Real-Time Discord Communication: Discord Message Send

These rules are ALWAYS ACTIVE for all Discord adapter components that send messages to channels, threads, or users in reminder.py and bot_manager.py.

### Rules

- **R-DISCORD-001** MUST: All Discord message send operations MUST use async/await syntax (await channel.send(), await thread.send(), await user.send())
- **R-DISCORD-002** MUST: HTTP client operations to TLT service endpoints MUST use async patterns with aiohttp.AsyncClient
- **R-DISCORD-003** MUST: Discord event handlers (on_ready, on_guild_join, on_reaction_add) MUST be implemented as async functions
- **R-DISCORD-004** MUST: Reminder scheduling and delivery functions MUST use await for all I/O operations
- **R-DISCORD-005** MUST: State monitoring and event update polling MUST use async/await syntax
- **R-DISCORD-006** SHOULD: Use asyncio.gather() to parallelize independent async operations such as sending multiple messages or querying multiple endpoints
- **R-DISCORD-007** SHOULD: Implement proper exception handling within async functions using try/except blocks with logging.getLogger(__name__)
- **R-DISCORD-008** MAY: Synchronous utility functions that do not perform I/O are exempt from async requirements
- **R-DISCORD-009** MAY: Initialization code that runs before the event loop starts (EXC-001) may use synchronous patterns with documented justification

### Verify

```bash
# Count async send operations
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Count non-awaited send operations (should be zero or only in comments/exceptions)
grep -r '\.send(' monorepo/tlt/adapters/discord_adapter/ | grep -v 'await' | grep -v '#' | wc -l

# Run async integration tests
python -m pytest monorepo/tlt/adapters/discord_adapter/tests/ -k 'async' -v

# Check for blocking operations in async contexts
python -m pylint --disable=all --enable=not-async-context-manager,await-outside-async monorepo/tlt/adapters/discord_adapter/
```

**Accept when:**
- All .send() calls in Discord adapter files are preceded by await keyword
- No synchronous blocking calls to Discord API are present in async functions
- Async integration tests pass without event loop blocking warnings
- Static analysis with pylint async rules reports no violations
- All exceptions to async/await usage are documented with inline comments and tech lead approval

<enforcement>
Claude Code MUST NOT skip or defer verification. All Discord message send operations MUST comply with R-DISCORD-001 through R-DISCORD-007. Violations block merge until remediated or formally excepted per the exception process.
</enforcement>