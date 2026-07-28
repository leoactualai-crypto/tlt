# Standardize Ephemeral Message Responses for Discord Interaction Feedback: Event Selection Interfaces

These rules are ALWAYS ACTIVE for all Discord adapter command handlers in `monorepo/tlt/adapters/discord_adapter/commands/` and all interaction response methods used within them.

### Rules

- **R-EPHEMERAL-001** SHOULD: Event selection interfaces (embeds with views) SHOULD use `ephemeral=True` to keep selection UI private to the command initiator.

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
- All `interaction.response.send_message()` calls in command handlers include an explicit `ephemeral` parameter (either `True` or `False`)
- Integration tests validate `ephemeral=True` for error states, empty states, and success confirmations
- Code review checklist confirms ephemeral messaging pattern compliance before merge
- Non-ephemeral responses are documented with explicit justification in handler docstring and pull request description

<enforcement>
Claude Code MUST NOT skip or defer verification of ephemeral parameter presence in all interaction response calls. Violations must be flagged during code review and CI pipeline checks must fail if explicit ephemeral parameters are missing without documented exception.
</enforcement>