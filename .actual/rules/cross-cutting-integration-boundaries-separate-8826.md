# Adopt Async Real-Time Messaging for Discord Integration Boundaries: Integration Boundaries Separate

These rules are ALWAYS ACTIVE for all Discord adapter implementations and real-time messaging boundaries within the TLT service integration layer, including all files in `monorepo/tlt/adapters/discord_adapter/` and FastAPI router endpoints that trigger Discord messaging operations.

### Rules

- **R-DISCORD-001** MUST: Integration boundaries MUST separate HTTP request-response patterns (FastAPI routers) from real-time event-driven patterns (Discord bot lifecycle hooks) with explicit async coordination.
- **R-DISCORD-002** MUST: All Discord messaging operations (channel.send, thread.send, user.send) MUST be prefixed with the 'await' keyword in adapter modules.
- **R-DISCORD-003** MUST: All public API contracts in Discord adapter modules MUST use 'async def' function signatures.
- **R-DISCORD-004** MUST: External HTTP client usage MUST employ AsyncClient or ClientSession patterns rather than synchronous requests.
- **R-DISCORD-005** MUST: Pydantic BaseModel schemas MUST be defined for all data structures crossing integration boundaries (ReminderCreate, ReminderResponse, etc.).
- **R-DISCORD-006** MUST: All async tasks MUST be wrapped with exception handlers; unhandled exceptions in async tasks (reminder checks, state polling) are not permitted.
- **R-DISCORD-007** MUST: Graceful shutdown handlers MUST await pending tasks and close async resources (HTTP clients, Discord connections) to prevent resource leaks.
- **R-DISCORD-008** SHOULD: Use asyncio.create_task() for fire-and-forget operations like reminder scheduling, and await directly for operations requiring immediate results.
- **R-DISCORD-009** SHOULD: Configure aiohttp.ClientSession or httpx.AsyncClient with connection pooling and timeout settings to prevent resource exhaustion.
- **R-DISCORD-010** MAY: Synchronous wrapper functions are permitted only for testing or CLI tooling that mocks Discord interactions, and MUST be documented with inline comments explaining the deviation.

### Verify

```bash
# Verify await usage on Discord send operations
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify async def usage in key functions
grep -r 'async def' monorepo/tlt/adapters/discord_adapter/ | grep -E '(create_reminder|schedule_reminder|setup_hook|on_ready)' | wc -l

# Verify async HTTP client patterns
grep -r 'AsyncClient\|ClientSession' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify Pydantic BaseModel schemas at boundaries
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -E '(ReminderCreate|ReminderResponse)' | wc -l
```

**Accept when:**
- All Discord messaging operations (channel.send, thread.send, user.send) are prefixed with 'await' keyword in adapter modules
- All public API contracts in Discord adapter modules use 'async def' function signatures
- External HTTP client usage shows AsyncClient or ClientSession patterns rather than synchronous requests
- Pydantic BaseModel schemas are defined for data structures crossing integration boundaries (ReminderCreate, ReminderResponse)
- No blocking operations (synchronous file I/O, blocking HTTP calls) are detected in async code paths without documented exceptions
- All async tasks include exception handlers and dead letter logging for failed operations

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for Discord adapter implementations. Violations MUST be caught during code review and CI pipeline checks before merge. Runtime monitoring of event loop lag metrics MUST be configured to detect blocking operations in production.
</enforcement>