# Use Async Message Send for Real-Time Discord Communication: Components Use Asyncio

These rules are ALWAYS ACTIVE for all Discord adapter components that send messages to channels, threads, or users, including all Discord event handlers, reminder scheduling functions, and HTTP client operations to service endpoints.

### Rules

- **R-ASYNC-001** MAY: Components MAY use asyncio task loops (e.g., `reminder_check_task.start()`) for periodic background operations.
- **R-ASYNC-002** MUST: All Discord message send operations must use `await` with `channel.send()`, `thread.send()`, or `user.send()` methods.
- **R-ASYNC-003** MUST: HTTP client operations must use `aiohttp.AsyncClient` with async context managers (`async with`) for proper resource cleanup.
- **R-ASYNC-004** SHOULD: Use `asyncio.gather()` to parallelize independent async operations such as sending multiple messages or querying multiple endpoints.
- **R-ASYNC-005** MUST: Implement proper exception handling within async functions using try/except blocks and log errors using the logging module (`logging.getLogger(__name__)`).
- **R-ASYNC-006** MUST NOT: Introduce blocking calls that degrade performance in async contexts.
- **R-ASYNC-007** MUST NOT: Use synchronous blocking calls to Discord API in async functions without documented exception approval.

### Verify

```bash
# Count await usage with send operations
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Count send operations without await (should be minimal/zero in async contexts)
grep -r '\.send(' monorepo/tlt/adapters/discord_adapter/ | grep -v 'await' | grep -v '#' | wc -l

# Run async integration tests
python -m pytest monorepo/tlt/adapters/discord_adapter/tests/ -k 'async' -v

# Check for blocking operations using pylint
pylint --disable=all --enable=not-async-context-manager,await-outside-async monorepo/tlt/adapters/discord_adapter/
```

**Accept when:**
- All `.send()` calls in Discord adapter files are preceded by `await` keyword
- No synchronous blocking calls to Discord API are present in async functions
- Async integration tests pass without event loop blocking warnings
- Static analysis using pylint async rules reports no violations
- Performance regression tests do not flag event loop blocking

<enforcement>
Claude Code MUST NOT skip or defer verification. Code review MUST block merge if blocking I/O is found in async functions without documented exception approval from tech lead. CI pipeline MUST fail if synchronous Discord API calls are detected in async contexts.
</enforcement>