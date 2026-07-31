# Adopt Async Real-Time Messaging for Discord Integration Boundaries: Discord Adapter Modules

These rules are ALWAYS ACTIVE for all Discord adapter modules in `monorepo/tlt/adapters/discord_adapter/`, FastAPI router endpoints that trigger Discord messaging operations, bot lifecycle hooks, reminder scheduling workflows, and state synchronization tasks between Discord and TLT service backend.

### Rules

- **R-DISCORD-001** MUST: Discord adapter modules MUST implement async function signatures for all public API contracts including `create_reminder`, `schedule_reminder`, `get_reminder`, `list_reminders`, `delete_reminder`, `setup_hook`, `on_ready`, `on_guild_join`, `on_guild_remove`, and `on_reaction_add`.
- **R-DISCORD-002** MUST: All Discord messaging operations (`channel.send()`, `thread.send()`, `user.send()`) MUST be prefixed with the `await` keyword in adapter modules.
- **R-DISCORD-003** MUST: External HTTP client usage for backend API calls MUST use async patterns (`aiohttp.ClientSession` or `httpx.AsyncClient`) rather than synchronous requests.
- **R-DISCORD-004** MUST: Pydantic `BaseModel` schemas MUST be defined for all data structures crossing integration boundaries (e.g., `ReminderCreate`, `ReminderResponse`).
- **R-DISCORD-005** SHOULD: Use `asyncio.create_task()` for fire-and-forget operations like reminder scheduling, and `await` directly for operations requiring immediate results.
- **R-DISCORD-006** SHOULD: Configure async HTTP clients with connection pooling and timeout settings to prevent resource exhaustion during backend API calls.
- **R-DISCORD-007** SHOULD: Implement graceful shutdown handlers that await pending tasks and close async resources to prevent resource leaks.
- **R-DISCORD-008** MAY: Synchronous wrapper functions are permitted for testing or CLI tooling that mocks Discord interactions, provided they are documented and isolated from production code paths.

### Verify

```bash
# Count await patterns on Discord send operations
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Count async def signatures for public API contracts
grep -r 'async def' monorepo/tlt/adapters/discord_adapter/ | grep -E '(create_reminder|schedule_reminder|setup_hook|on_ready)' | wc -l

# Verify async HTTP client usage
grep -r 'AsyncClient\|ClientSession' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify Pydantic BaseModel schemas at boundaries
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -E '(ReminderCreate|ReminderResponse)' | wc -l
```

**Accept when:**
- All Discord messaging operations (`channel.send`, `thread.send`, `user.send`) are prefixed with `await` keyword in adapter modules.
- All public API contracts in Discord adapter modules use `async def` function signatures.
- External HTTP client usage shows `AsyncClient` or `ClientSession` patterns rather than synchronous requests.
- Pydantic `BaseModel` schemas are defined for data structures crossing integration boundaries (`ReminderCreate`, `ReminderResponse`).
- Code review checks using pylint-asyncio detect no blocking operations in async functions.
- Event loop lag metrics remain within acceptable thresholds in production.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for Discord adapter implementations. Violations detected by CI pipeline grep-based verification commands, pylint-asyncio checks, or runtime monitoring MUST be remediated before merge. Exceptions require tech lead approval and must be documented with justification.
</enforcement>