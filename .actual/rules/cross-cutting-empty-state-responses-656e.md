# Standardize Ephemeral Message Responses for Discord Interaction Feedback: Empty State Responses

These rules are ALWAYS ACTIVE for all Discord adapter command handlers in `monorepo/tlt/adapters/discord_adapter/commands/` and all interaction response methods within scope.

### Rules

- **R-EPHEMERAL-001** MUST: Empty state responses (no events found, no data available) MUST be delivered as ephemeral messages via `interaction.response.send_message(ephemeral=True)`.
- **R-EPHEMERAL-002** MUST: All `interaction.response.send_message()` calls in command handlers MUST include an explicit `ephemeral` parameter (either `True` or `False`).
- **R-EPHEMERAL-003** SHOULD: Error handling and success confirmation flows SHOULD follow the ephemeral messaging pattern established in the codebase.
- **R-EPHEMERAL-004** MAY: Public event announcements or channel-wide notifications MAY use non-ephemeral responses when explicitly required by feature specification (EXC-001).

### Verify

```bash
# Check for interaction.response.send_message calls without explicit ephemeral parameter
grep -r 'interaction.response.send_message' monorepo/tlt/adapters/discord_adapter/commands/ | grep -v 'ephemeral=True' | grep -v 'ephemeral=False'

# Count consistent ephemeral=True usage across handlers
grep -r 'send_message.*ephemeral=True' monorepo/tlt/adapters/discord_adapter/commands/ | wc -l

# Run ephemeral message integration tests
python -m pytest monorepo/tlt/adapters/discord_adapter/commands/ -k 'test_ephemeral' -v
```

**Accept when:**
- All `interaction.response.send_message()` calls in command handlers include explicit `ephemeral` parameter
- Integration tests validate `ephemeral=True` for error states, empty states, and success confirmations
- Code review checklist confirms ephemeral messaging pattern compliance before merge
- Any non-ephemeral responses are documented with explicit justification in handler docstring and pull request description

<enforcement>
Claude Code MUST NOT skip or defer verification. All interaction response calls must include explicit ephemeral parameters. Violations block merge unless product owner and engineering lead approve documented exceptions.
</enforcement>