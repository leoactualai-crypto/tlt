# Adopt Async Real-Time Messaging for Discord Bot Communication: External Http Client

These rules are ALWAYS ACTIVE for all Discord adapter components within the TLT monorepo that interact with real-time messaging boundaries, including bot_manager.py, reminder.py, Discord event handlers, and FastAPI route handlers that interact with Discord bot instances.

### Rules

- **R-ASYNC-001** MUST: External HTTP client operations to TLT services MUST use async HTTP clients (httpx.AsyncClient, aiohttp) within async context managers.
- **R-ASYNC-002** MUST: All Discord event handlers (on_ready, on_guild_join, on_reaction_add, setup_hook) MUST be declared as async functions.
- **R-ASYNC-003** MUST: All Discord messaging operations (channel.send(), user.send(), thread.send()) MUST be preceded by the await keyword.
- **R-ASYNC-004** MUST: No synchronous blocking libraries (requests, time.sleep, synchronous file I/O) MUST be imported or used in async contexts.
- **R-ASYNC-005** MUST: All external HTTP requests MUST include timeout parameters to prevent indefinite hangs that would block the event loop.
- **R-ASYNC-006** SHOULD: Use discord.py's @tasks.loop() decorator for periodic background operations like reminder checks and state monitoring, ensuring proper error handling and restart logic.
- **R-ASYNC-007** SHOULD: Wrap all async tasks with try-except blocks and log exceptions with full stack traces to prevent silent failures.
- **R-ASYNC-008** SHOULD: Use asyncio.Lock for critical sections accessing shared state (active_reminders dictionary) from multiple async contexts.

### Verify

```bash
# Verify all event handlers are async
grep -r 'def on_\|def setup_hook' monorepo/tlt/adapters/discord_adapter/ | grep -v 'async def' && echo 'FAIL: Found non-async event handlers' || echo 'PASS: All event handlers are async'

# Verify all send operations are awaited
grep -r '\.send(' monorepo/tlt/adapters/discord_adapter/ | grep -v 'await' && echo 'FAIL: Found non-awaited send operations' || echo 'PASS: All send operations are awaited'

# Verify no blocking operations are present
grep -r 'import requests\|time\.sleep\|open(' monorepo/tlt/adapters/discord_adapter/*.py && echo 'FAIL: Found blocking operations' || echo 'PASS: No blocking operations detected'
```

**Accept when:**
- All Discord event handlers (on_ready, on_guild_join, on_reaction_add, setup_hook) are declared as async functions
- All Discord messaging operations (channel.send(), user.send(), thread.send()) are preceded by await keyword
- No synchronous blocking libraries (requests, time.sleep, synchronous file I/O) are imported or used in async contexts
- All external HTTP client operations use async clients (httpx.AsyncClient, aiohttp) within async context managers
- All external HTTP requests include timeout parameters

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks block commits containing synchronous blocking operations in async contexts. Code review requires mandatory changes before merge if async patterns are violated. CI failures on integration tests trigger automatic PR comments with remediation guidance.
</enforcement>