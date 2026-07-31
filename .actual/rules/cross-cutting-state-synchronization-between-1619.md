# Adopt Async Real-Time Messaging for Discord Integration Boundaries: State Synchronization Between

These rules are ALWAYS ACTIVE for all Discord adapter implementations and real-time messaging boundaries within the TLT service integration layer, including all files in `monorepo/tlt/adapters/discord_adapter/`, FastAPI router endpoints that trigger Discord messaging operations, bot lifecycle hooks, reminder scheduling workflows, and state synchronization tasks between Discord and TLT service backend.

### Rules

- **R-DISCORD-001** SHOULD: State synchronization between Discord and TLT service SHOULD implement periodic polling tasks with configurable intervals (STATE_QUERY_INTERVAL) using async task scheduling.
- **R-DISCORD-002** MUST: All Discord messaging operations (channel.send, thread.send, user.send) MUST be prefixed with the 'await' keyword in adapter modules.
- **R-DISCORD-003** MUST: All public API contracts in Discord adapter modules MUST use 'async def' function signatures.
- **R-DISCORD-004** MUST: External HTTP client usage MUST employ AsyncClient or ClientSession patterns rather than synchronous requests.
- **R-DISCORD-005** MUST: Pydantic BaseModel schemas MUST be defined for all data structures crossing integration boundaries (ReminderCreate, ReminderResponse).
- **R-DISCORD-006** MUST: All async tasks (reminder checks, state polling) MUST be wrapped with exception handlers to prevent silent failures.
- **R-DISCORD-007** MUST: Graceful shutdown handlers MUST await pending tasks and close async resources (HTTP clients, Discord connections) to prevent resource leaks.
- **R-DISCORD-008** MAY: Synchronous wrapper functions MAY be required for testing or CLI tooling that mocks Discord interactions (EXC-001).

### Verify

```bash
# Verify await usage on Discord send operations
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify async def usage in key adapter functions
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
- All async tasks include exception handlers and dead letter logging for failed operations
- Graceful shutdown handlers are implemented to close async resources

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated code review checks using pylint-asyncio MUST detect blocking operations in async functions. CI pipeline MUST execute grep-based verification commands to ensure async patterns are present in adapter modules. Manual code review MUST verify async/await usage at integration boundaries. Runtime monitoring of event loop lag metrics MUST detect blocking operations in production. Violations result in CI pipeline failure, code review blocks, runtime alerts, and quarterly architecture review audits.
</enforcement>