# Enforce Input Validation and HTTP Exception Handling in Discord Integration Endpoints: Get Endpoints Returning

These rules are ALWAYS ACTIVE for all HTTP endpoint handlers in the Discord adapter that access bot state, query event reactions, or call Discord API operations.

### Rules

- **R-DISCORD-001** MUST: Validate message_id existence in bot.active_events before proceeding with Discord API calls or state mutations.
- **R-DISCORD-002** MUST: Use Pydantic BaseModel for all request body validation; define typed fields and let automatic parsing prevent malformed data from reaching business logic.
- **R-DISCORD-003** MUST: Raise HTTPException with status_code=404 for missing resources (events, guild, channel, message, user) and status_code=500 for unhandled exceptions.
- **R-DISCORD-004** SHOULD: Use dict.get() with default values to safely access optional nested structures in event state dictionaries.
- **R-DISCORD-005** SHOULD: Defensively initialize nested state structures before mutation (e.g., if 'reactions' not in event: event['reactions'] = {}).
- **R-DISCORD-006** SHOULD: Log validation failures at info level for user-triggered conditions and error level for system exceptions.
- **R-DISCORD-007** MAY: Implement retry logic with exponential backoff for transient Discord API failures and circuit breaker patterns for degraded availability.

### Verify

```bash
# Verify Pydantic BaseModel usage for request validation
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -v '__pycache__'

# Verify HTTPException is raised for error conditions
grep -r 'raise HTTPException' monorepo/tlt/adapters/discord_adapter/rsvp.py | wc -l

# Verify message_id validation pattern is applied
grep -r 'if.*not in bot.active_events' monorepo/tlt/adapters/discord_adapter/rsvp.py
```

**Accept when:**
- All HTTP endpoint handlers validate message_id existence in bot.active_events before Discord API calls
- Request bodies use Pydantic BaseModel validation and handlers raise HTTPException for missing resources (404) and exceptions (500)
- Nested state structures are defensively initialized before mutation operations
- dict.get() is used for safe access to optional nested event state
- Validation failures are logged at appropriate levels (info for user-triggered, error for system exceptions)

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All new HTTP endpoints accessing bot state or Discord API must pass the verification commands and acceptance criteria before merge.
</enforcement>