# Adopt Async Real-Time Messaging for Discord Bot Communication: Discord Interactions That

These rules are ALWAYS ACTIVE for all Discord adapter components within the TLT monorepo that interact with real-time messaging boundaries, including bot_manager.py, reminder.py, event handlers, and FastAPI route handlers that interact with Discord bot instances.

### Rules

- **R-DISCORD-ASYNC-001** MUST: All Discord API interactions that involve I/O operations (channel.send(), user.send(), thread.send()) MUST use async/await syntax.
- **R-DISCORD-ASYNC-002** MUST: All Discord event handlers (on_ready, on_guild_join, on_reaction_add, setup_hook) MUST be declared as async functions.
- **R-DISCORD-ASYNC-003** MUST: All external HTTP client operations to TLT services MUST use async clients (httpx.AsyncClient, aiohttp) within async context managers.
- **R-DISCORD-ASYNC-004** MUST: Background tasks for reminder scheduling and state monitoring MUST use async/await patterns and discord.py's @tasks.loop() decorator.
- **R-DISCORD-ASYNC-005** MUST: FastAPI route handlers that interact with Discord bot instances MUST be declared as async functions.
- **R-DISCORD-ASYNC-006** SHOULD: Wrap all async tasks with try-except blocks and log exceptions with full stack traces to prevent silent failures.
- **R-DISCORD-ASYNC-007** SHOULD: Use asyncio.Lock for critical sections accessing shared state (active_reminders dictionary) from multiple async contexts.
- **R-DISCORD-ASYNC-008** SHOULD: Add timeout parameters to all external HTTP requests to prevent indefinite hangs that would block the event loop.
- **R-DISCORD-ASYNC-009** MAY: For CPU-bound operations that must run synchronously, use asyncio.to_thread() or loop.run_in_executor() to offload work to thread pools without blocking the event loop.

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
- Background task operations use discord.py's @tasks.loop() decorator with proper error handling
- Shared state access is protected with asyncio.Lock where necessary

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for Discord adapter code. Pre-commit hooks MUST block commits containing synchronous blocking operations in async contexts. Code review MUST verify async/await usage before merge. CI pipeline integration tests MUST measure event loop responsiveness under concurrent load.
</enforcement>