# Standardize Ephemeral Message Responses for Discord Interaction Feedback: Command Handlers Use

These rules are ALWAYS ACTIVE for all Discord adapter command handlers in `monorepo/tlt/adapters/discord_adapter/commands/` that use `interaction.response.send_message()` for user feedback.

### Rules

- **R-EPHEMERAL-001** MUST: Command handlers MUST use `interaction.response.send_message()` for initial responses and follow-up methods for subsequent messages.
- **R-EPHEMERAL-002** MUST: All `interaction.response.send_message()` calls MUST include an explicit `ephemeral` parameter (either `ephemeral=True` or `ephemeral=False`).
- **R-EPHEMERAL-003** SHOULD: Command handlers SHOULD set `ephemeral=True` for error states, empty result states, and success confirmations to prevent channel clutter.
- **R-EPHEMERAL-004** SHOULD: Non-ephemeral responses SHOULD be documented with explicit justification in handler docstring and pull request description.
- **R-EPHEMERAL-005** MAY: Public event announcements or channel-wide notifications MAY use `ephemeral=False` when explicitly required by feature specification (EXC-001).

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
- Any non-ephemeral responses are documented with rationale in handler docstring and PR description

<enforcement>
Claude Code MUST NOT skip or defer verification. All interaction response calls must be audited for explicit ephemeral parameter presence. CI pipeline MUST fail if calls lack explicit ephemeral parameter. Code review MUST block merge if pattern is violated without documented justification.
</enforcement>