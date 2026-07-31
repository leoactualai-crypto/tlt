# Enforce Input Validation and HTTP Exception Handling in Discord Integration Endpoints: Endpoints Log Validation

These rules are ALWAYS ACTIVE for all HTTP endpoint handlers in the Discord adapter that access bot state or Discord API resources.

### Rules

- **R-DISCORD-LOG-001** SHOULD: Endpoints SHOULD log validation failures and exceptions using the module logger with appropriate severity levels (info for user actions, error for exceptions).

### Verify

```bash
# Verify Pydantic BaseModel usage for request validation
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -v '__pycache__'

# Verify HTTPException usage in endpoints
grep -r 'raise HTTPException' monorepo/tlt/adapters/discord_adapter/rsvp.py | wc -l

# Verify validation checks for active_events
grep -r 'if.*not in bot.active_events' monorepo/tlt/adapters/discord_adapter/rsvp.py
```

**Accept when:**
- All HTTP endpoint handlers validate message_id existence in bot.active_events before Discord API calls
- Request bodies use Pydantic BaseModel validation and handlers raise HTTPException for missing resources (404) and exceptions (500)
- Nested state structures are defensively initialized before mutation operations
- Validation failures are logged at info level for user-triggered conditions and error level for system exceptions

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All new HTTP endpoints accessing bot state or Discord API must include input validation, exception handling, and appropriate logging before merge.
</enforcement>