# Use Async Message Send for Real-Time Discord Communication: Concurrent Independent Operations

These rules are ALWAYS ACTIVE for all Discord adapter components that send messages to channels, threads, or users, including all Discord message send operations in reminder.py and bot_manager.py, HTTP client operations to TLT service endpoints, Discord event handlers, reminder scheduling and delivery functions, and state monitoring and event update polling.

### Rules

- **R-ASYNC-001** SHOULD: Concurrent independent operations (state queries, message sends) SHOULD use asyncio.gather or asyncio.create_task for parallelization.

### Verify

```bash
# Count await calls with .send() operations
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Count .send() calls without await (should be zero in async contexts)
grep -r '\.send(' monorepo/tlt/adapters/discord_adapter/ | grep -v 'await' | grep -v '#' | wc -l

# Run async integration tests
python -m pytest monorepo/tlt/adapters/discord_adapter/tests/ -k 'async' -v
```

**Accept when:**
- All .send() calls in Discord adapter files are preceded by await keyword
- No synchronous blocking calls to Discord API are present in async functions
- Async integration tests pass without event loop blocking warnings

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if blocking I/O is found in async functions without justification. CI pipeline MUST fail if synchronous Discord API calls are detected in async contexts.
</enforcement>