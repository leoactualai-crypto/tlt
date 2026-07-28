# Standardize Ephemeral Message Responses for Discord Interaction Feedback: Success Confirmation Messages

These rules are ALWAYS ACTIVE for all Discord adapter command handlers in `monorepo/tlt/adapters/discord_adapter/commands/` that use `interaction.response.send_message()` and related response methods for error handling, empty state handling, and success confirmation flows.

### Rules

- **R-DISCORD-001** MUST: Success confirmation messages MUST use `ephemeral=True` to provide private feedback to the command initiator.
- **R-DISCORD-002** MUST: All `interaction.response.send_message()` calls in command handlers MUST include an explicit `ephemeral` parameter (either `True` or `False`).
- **R-DISCORD-003** MUST: Error state messages MUST use `ephemeral=True` to prevent channel clutter.
- **R-DISCORD-004** MUST: Empty state messages MUST use `ephemeral=True` to provide private feedback.
- **R-DISCORD-005** SHOULD: Create a base handler class or utility function that wraps `interaction.response.send_message()` with `ephemeral=True` as the default parameter.
- **R-DISCORD-006** SHOULD: Document the ephemeral messaging pattern in the Discord adapter README and developer onboarding materials.

### Verify

```bash
# Check for interaction.response.send_message calls without explicit ephemeral parameter
grep -r 'interaction.response.send_message' monorepo/tlt/adapters/discord_adapter/commands/ | grep -v 'ephemeral=True' | grep -v 'ephemeral=False'

# Count consistent usage of ephemeral=True across handlers
grep -r 'send_message.*ephemeral=True' monorepo/tlt/adapters/discord_adapter/commands/ | wc -l

# Run ephemeral message tests
python -m pytest monorepo/tlt/adapters/discord_adapter/commands/ -k 'test_ephemeral' -v
```

**Accept when:**
- All `interaction.response.send_message()` calls in command handlers include explicit `ephemeral` parameter
- Integration tests validate `ephemeral=True` for error states, empty states, and success confirmations
- Code review checklist confirms ephemeral messaging pattern compliance before merge
- Any non-ephemeral message usage is documented with explicit justification in handler docstring and pull request description

<enforcement>
Claude Code MUST NOT skip or defer verification. All interaction response calls must be audited for explicit ephemeral parameter presence. CI pipeline MUST fail if `interaction.response.send_message()` calls lack explicit ephemeral parameter. Code review MUST block merge if ephemeral messaging pattern is violated without documented justification.
</enforcement>