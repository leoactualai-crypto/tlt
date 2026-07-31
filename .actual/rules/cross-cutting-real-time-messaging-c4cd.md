# Adopt Async Real-Time Messaging for Discord Integration Boundaries: Real Time Messaging

These rules are ALWAYS ACTIVE for all Discord adapter implementations and real-time messaging boundaries within the TLT service integration layer, including all files in `monorepo/tlt/adapters/discord_adapter/`, FastAPI router endpoints that trigger Discord messaging operations, bot lifecycle hooks, reminder scheduling workflows, and state synchronization tasks.

### Rules

- **R-DISCORD-001** MUST: All real-time messaging operations at Discord integration boundaries MUST use async/await patterns with explicit await calls on send operations (channel.send(), thread.send(), user.send()).

### Verify

```bash
# Count await calls on Discord send operations
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify async def signatures for key reminder and lifecycle functions
grep -r 'async def' monorepo/tlt/adapters/discord_adapter/ | grep -E '(create_reminder|schedule_reminder|setup_hook|on_ready)' | wc -l

# Verify async HTTP client usage (AsyncClient or ClientSession)
grep -r 'AsyncClient\|ClientSession' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify Pydantic BaseModel schemas at integration boundaries
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -E '(ReminderCreate|ReminderResponse)' | wc -l
```

**Accept when:**
- All Discord messaging operations (channel.send, thread.send, user.send) are prefixed with 'await' keyword in adapter modules
- All public API contracts in Discord adapter modules use 'async def' function signatures
- External HTTP client usage shows AsyncClient or ClientSession patterns rather than synchronous requests
- Pydantic BaseModel schemas are defined for data structures crossing integration boundaries (ReminderCreate, ReminderResponse)

<enforcement>
Claude Code MUST NOT skip or defer verification. All four verification commands must return non-zero results indicating async patterns are present. CI pipeline fails if verification commands do not meet acceptance criteria thresholds. Code review blocks merge if blocking operations are detected in async code paths without justification.
</enforcement>