# Adopt Async Real-Time Messaging for Discord Bot Communication: Event Handlers Discord

These rules are ALWAYS ACTIVE for all Discord adapter components within the TLT monorepo that interact with real-time messaging boundaries, including bot_manager.py, reminder.py, and FastAPI route handlers that interact with Discord bot instances.

### Rules

- **R-DISCORD-001** MUST: Event handlers for Discord lifecycle events (setup_hook, on_ready, on_guild_join, on_guild_remove, on_reaction_add) MUST be declared as async functions.
- **R-DISCORD-002** MUST: All Discord messaging operations (channel.send(), user.send(), thread.send()) MUST be preceded by the await keyword.
- **R-DISCORD-003** MUST: All external HTTP client operations MUST use async clients (httpx.AsyncClient, aiohttp) within async context managers.
- **R-DISCORD-004** MUST: No synchronous blocking libraries (requests, time.sleep, synchronous file I/O) MUST be imported or used in async contexts.
- **R-DISCORD-005** SHOULD: Use discord.py's @tasks.loop() decorator for periodic background operations like reminder checks and state monitoring, ensuring proper error handling and restart logic.
- **R-DISCORD-006** SHOULD: Wrap all httpx.AsyncClient usage in async context managers (async with httpx.AsyncClient() as client:) to ensure proper connection cleanup.
- **R-DISCORD-007** SHOULD: For CPU-bound operations that must run synchronously, use asyncio.to_thread() or loop.run_in_executor() to offload work to thread pools without blocking the event loop.
- **R-DISCORD-008** SHOULD: Implement structured logging with correlation IDs to trace async operations across event handlers, background tasks, and API routes.
- **R-DISCORD-009** SHOULD: Add timeout parameters to all external HTTP requests (httpx.AsyncClient(timeout=...)) to prevent indefinite hangs that would block the event loop.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks block commits containing synchronous blocking operations in async contexts. Code review requires mandatory changes before merge if async patterns are violated. CI failures on integration tests trigger automatic PR comments with remediation guidance.
</enforcement>