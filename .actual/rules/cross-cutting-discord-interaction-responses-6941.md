# Standardize Ephemeral Message Responses for Discord Interaction Feedback: Discord Interaction Responses

These rules are ALWAYS ACTIVE for all Discord adapter command handlers in `monorepo/tlt/adapters/discord_adapter/commands/` and all interaction response methods within scope.

### Rules

- **R-DISCORD-001** MUST: All Discord interaction responses for error states MUST use `ephemeral=True` parameter to prevent channel clutter.
- **R-DISCORD-002** MUST: All `interaction.response.send_message()` calls in command handlers MUST include an explicit ephemeral parameter (either `ephemeral=True` or `ephemeral=False` with documented justification).
- **R-DISCORD-003** SHOULD: Create a base handler class or utility function that wraps `interaction.response.send_message()` with `ephemeral=True` as the default parameter.
- **R-DISCORD-004** SHOULD: Document the ephemeral messaging pattern in the Discord adapter README and developer onboarding materials.

### Verify

```bash
# Find all interaction.response.send_message calls without explicit ephemeral parameter
grep -r 'interaction.response.send_message' monorepo/tlt/adapters/discord_adapter/commands/ | grep -v 'ephemeral=True' | grep -v 'ephemeral=False'

# Count consistent usage of ephemeral=True across handlers
grep -r 'send_message.*ephemeral=True' monorepo/tlt/adapters/discord_adapter/commands/ | wc -l

# Run ephemeral message tests
python -m pytest monorepo/tlt/adapters/discord_adapter/commands/ -k 'test_ephemeral' -v
```

**Accept when:**
- All `interaction.response.send_message()` calls in command handlers include explicit ephemeral parameter
- Integration tests validate `ephemeral=True` for error states, empty states, and success confirmations
- Code review checklist confirms ephemeral messaging pattern compliance before merge
- Any non-ephemeral responses are documented with explicit justification in handler docstring

<enforcement>
Clause Code MUST NOT skip or defer verification. CI pipeline MUST fail if `interaction.response.send_message()` calls lack explicit ephemeral parameter. Code review MUST block merge if ephemeral messaging pattern is violated without documented justification. Post-merge violations trigger technical debt ticket for remediation.
</enforcement>