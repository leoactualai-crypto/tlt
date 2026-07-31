# Adopt Async Real-Time Messaging for Discord Integration Boundaries: External Client Interactions

These rules are ALWAYS ACTIVE for all Discord adapter implementations and real-time messaging boundaries within the TLT service integration layer, including all files in `monorepo/tlt/adapters/discord_adapter/`, FastAPI router endpoints that trigger Discord messaging operations, bot lifecycle hooks, reminder scheduling workflows, and state synchronization tasks between Discord and TLT service backend.

### Rules

- **R-DISCORD-ASYNC-001** MUST: External client interactions with the TLT service backend MUST use async HTTP clients (aiohttp.ClientSession, httpx.AsyncClient) to prevent blocking the Discord event loop.

### Verify

```bash
# Verify all Discord messaging operations use await keyword
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify async function definitions for key operations
grep -r 'async def' monorepo/tlt/adapters/discord_adapter/ | grep -E '(create_reminder|schedule_reminder|setup_hook|on_ready)' | wc -l

# Verify async HTTP client usage
grep -r 'AsyncClient\|ClientSession' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify Pydantic BaseModel schemas at boundaries
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -E '(ReminderCreate|ReminderResponse)' | wc -l
```

**Accept when:**
- All Discord messaging operations (channel.send, thread.send, user.send) are prefixed with 'await' keyword in adapter modules
- All public API contracts in Discord adapter modules use 'async def' function signatures
- External HTTP client usage shows AsyncClient or ClientSession patterns rather than synchronous requests
- Pydantic BaseModel schemas are defined for data structures crossing integration boundaries (ReminderCreate, ReminderResponse)

<enforcement>
Clause Code MUST NOT skip or defer verification. Violations trigger CI pipeline failure, code review blocks, and runtime alerts if event loop lag exceeds threshold. Exceptions require tech lead approval with documented sync-to-async bridge patterns.
</enforcement>