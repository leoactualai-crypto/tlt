# Standardize Ephemeral Message Responses for Discord Interaction Feedback: Public Event Announcements

These rules are ALWAYS ACTIVE for all Discord adapter command handlers in `monorepo/tlt/adapters/discord_adapter/commands/` and all interaction response methods across the codebase.

### Rules

- **R-EPHEMERAL-001** MUST: All `interaction.response.send_message()` calls in command handlers MUST include an explicit `ephemeral` parameter (either `ephemeral=True` or `ephemeral=False` with documented justification).
- **R-EPHEMERAL-002** MUST: Error states, empty result states, and success confirmations MUST use `ephemeral=True` to prevent channel clutter and maintain private user feedback.
- **R-EPHEMERAL-003** MAY: Public event announcements or shared event information MAY use non-ephemeral messages when broadcasting to the channel is the intended behavior and explicitly required by feature specification.
- **R-EPHEMERAL-004** MUST: Modal submission handlers and view interaction callbacks MUST follow the same ephemeral messaging pattern as command handlers.
- **R-EPHEMERAL-005** MUST: Any non-ephemeral message response MUST be documented in the handler docstring with explicit rationale and approved by product owner.

### Verify

```bash
# Find all interaction.response.send_message calls without explicit ephemeral parameter
grep -r 'interaction.response.send_message' monorepo/tlt/adapters/discord_adapter/commands/ | grep -v 'ephemeral=True' | grep -v 'ephemeral=False'

# Count consistent ephemeral=True usage across handlers
grep -r 'send_message.*ephemeral=True' monorepo/tlt/adapters/discord_adapter/commands/ | wc -l

# Run ephemeral message integration tests
python -m pytest monorepo/tlt/adapters/discord_adapter/commands/ -k 'test_ephemeral' -v

# Verify all handlers have explicit ephemeral parameter
grep -r 'interaction.response.send_message' monorepo/tlt/adapters/discord_adapter/commands/ | grep -c 'ephemeral='
```

**Accept when:**
- All `interaction.response.send_message()` calls in command handlers include explicit `ephemeral` parameter
- Integration tests validate `ephemeral=True` for error states, empty states, and success confirmations
- Code review checklist confirms ephemeral messaging pattern compliance before merge
- Any non-ephemeral responses are documented with rationale in handler docstring and approved by product owner
- Modal submission handlers and view interaction callbacks follow the same ephemeral pattern

<enforcement>
Claude Code MUST NOT skip or defer verification. All interaction response calls MUST include explicit ephemeral parameter. CI pipeline MUST fail if interaction.response.send_message() calls lack explicit ephemeral parameter. Code review MUST block merge if ephemeral messaging pattern is violated without documented justification and product owner approval.
</enforcement>