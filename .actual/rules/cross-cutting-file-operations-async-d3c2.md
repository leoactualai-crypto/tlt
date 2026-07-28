# Adopt Async Real-Time Messaging for Discord Bot Communication: File Operations Async

These rules are ALWAYS ACTIVE for all Discord adapter components within the TLT monorepo that interact with real-time messaging boundaries, including bot_manager.py, reminder.py, event handlers, and FastAPI route handlers that interact with Discord bot instances.

### Rules

- **R-ASYNC-001** SHOULD: File I/O operations in async contexts SHOULD use aiofiles to maintain non-blocking behavior.

### Verify

```bash
# Verify all Discord event handlers are async
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
- File I/O operations in async contexts use aiofiles instead of built-in open() or synchronous file operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for Discord adapter code within scope.
</enforcement>