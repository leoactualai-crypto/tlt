# Enforce Input Validation and HTTP Exception Handling in Discord Integration Endpoints: Reaction State Mutations

These rules are ALWAYS ACTIVE for all HTTP endpoint handlers in the Discord adapter that access bot state, perform Discord API calls, or mutate reaction state structures.

### Rules

- **R-DISCORD-REACTION-001** MUST: Reaction state mutations MUST initialize nested dictionary structures (event['reactions'], event['reactions'][emoji]) before accessing or modifying them
- **R-DISCORD-REACTION-002** MUST: All HTTP endpoint handlers MUST validate message_id existence in bot.active_events before proceeding with Discord API calls
- **R-DISCORD-REACTION-003** MUST: Request bodies MUST use Pydantic BaseModel validation (e.g., ReactionUpdate class) for type safety and automatic request parsing
- **R-DISCORD-REACTION-004** MUST: HTTP endpoints MUST raise HTTPException with status_code=404 for missing resources (events, guild, channel, message, user)
- **R-DISCORD-REACTION-005** MUST: HTTP endpoints MUST raise HTTPException with status_code=500 and sanitized error details for unhandled exceptions from Discord API calls
- **R-DISCORD-REACTION-006** MUST: Validation failures for user-triggered conditions (user not found, event not found) MUST be logged at info level
- **R-DISCORD-REACTION-007** MUST: System exceptions during validation or Discord API calls MUST be logged at error level

### Verify

```bash
# Verify Pydantic BaseModel definitions exist
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -v '__pycache__'

# Count HTTPException raises in rsvp.py
grep -r 'raise HTTPException' monorepo/tlt/adapters/discord_adapter/rsvp.py | wc -l

# Verify bot.active_events validation pattern
grep -r 'if.*not in bot.active_events' monorepo/tlt/adapters/discord_adapter/rsvp.py

# Verify nested dictionary initialization pattern
grep -r "if.*'reactions'.*not in event" monorepo/tlt/adapters/discord_adapter/rsvp.py
```

**Accept when:**
- All HTTP endpoint handlers validate message_id existence in bot.active_events before Discord API calls
- Request bodies use Pydantic BaseModel validation and handlers raise HTTPException for missing resources (404) and exceptions (500)
- Nested state structures are defensively initialized before mutation operations (event['reactions'], event['reactions'][emoji])
- Validation failures are logged at appropriate levels (info for user-triggered, error for system exceptions)
- Integration tests cover validation failure paths (missing events, missing Discord resources, malformed requests)

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All new HTTP endpoints accessing bot state or Discord API MUST satisfy R-DISCORD-REACTION-001 through R-DISCORD-REACTION-007 before merge. Integration test failures for validation paths block merge to main branch. Production incidents caused by unhandled exceptions trigger post-mortem review and enforcement.
</enforcement>