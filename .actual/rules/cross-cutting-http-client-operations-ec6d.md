# Use Async Message Send for Real-Time Discord Communication: Http Client Operations

These rules are ALWAYS ACTIVE for all Discord adapter components that send messages to channels, threads, or users, and for HTTP client operations to TLT service endpoints.

### Rules

- **R-DISCORD-001** SHOULD: HTTP client operations (aiohttp.AsyncClient) SHOULD use async context managers to ensure proper resource cleanup.
- **R-DISCORD-002** MUST: All Discord message send operations (.send() calls) must be preceded by the await keyword.
- **R-DISCORD-003** MUST: No synchronous blocking calls to Discord API are permitted in async functions.
- **R-DISCORD-004** SHOULD: Use asyncio.gather() to parallelize independent async operations such as sending multiple messages or querying multiple endpoints.
- **R-DISCORD-005** SHOULD: Implement proper exception handling within async functions using try/except blocks and log errors using the logging module.

### Verify

```bash
# Count await-prefixed send calls
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Count send calls without await (excluding comments)
grep -r '\.send(' monorepo/tlt/adapters/discord_adapter/ | grep -v 'await' | grep -v '#' | wc -l

# Run async integration tests
python -m pytest monorepo/tlt/adapters/discord_adapter/tests/ -k 'async' -v

# Check for async context manager usage
grep -r 'async with.*AsyncClient' monorepo/tlt/adapters/discord_adapter/ | wc -l
```

**Accept when:**
- All .send() calls in Discord adapter files are preceded by await keyword
- No synchronous blocking calls to Discord API are present in async functions
- Async integration tests pass without event loop blocking warnings
- HTTP client operations use async context managers (async with) for resource cleanup
- All async functions with I/O operations include try/except blocks with logging

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis via pylint async rules and integration tests measuring event loop responsiveness are mandatory before acceptance.
</enforcement>