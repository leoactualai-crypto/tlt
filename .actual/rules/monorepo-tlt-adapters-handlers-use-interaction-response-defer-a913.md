# Discord.py Interaction Response API for Command and UI Component Messaging: Handlers Use Interaction Response Defer Long

These rules are ALWAYS ACTIVE for all Discord slash command handlers, UI component callbacks (buttons, dropdowns, select menus, modals), command routers, and any code path that receives a `discord.Interaction` object and must respond to the user within the 3-second interaction token window.

### Rules

- **R-DISCORD-INTERACTION-001** MUST: Use `interaction.response.*` methods for all initial responses to Discord interactions within the 3-second token window; do not use direct `channel.send()` calls in interaction callback contexts.
- **R-DISCORD-INTERACTION-002** MAY: Use `interaction.response.defer()` for long-running operations that cannot complete within the 3-second interaction window, followed by `interaction.followup.send()` to deliver results after deferred response.
- **R-DISCORD-INTERACTION-003** MUST: Use ephemeral messaging (`ephemeral=True`) for all error responses, validation failures, and user-specific feedback in command handlers to avoid channel pollution.
- **R-DISCORD-INTERACTION-004** MUST: Implement async callback methods for all UI components (buttons, dropdowns, modals) that receive interaction objects.
- **R-DISCORD-INTERACTION-005** SHOULD: Use `interaction.response.edit_message()` to update existing message state rather than creating new messages in UI component callbacks.
- **R-DISCORD-INTERACTION-006** MUST: Wrap interaction response calls in try-except blocks to handle token expiration and API errors gracefully, providing fallback error messages through ephemeral responses when possible.
- **R-DISCORD-INTERACTION-007** SHOULD: Store intermediate state in the view or modal instance for multi-step workflows involving modals to maintain context across interaction callbacks.

### Verify

```bash
# Discover the project's test suite location and execute integration tests
# that validate interaction response patterns in command handlers
find . -type f -name '*test*.py' -o -name 'test_*' | head -5

# Discover the project's static analysis configuration and run type checking
# to verify all interaction.response API calls match the locked library version
find . -type f \( -name 'pyproject.toml' -o -name 'setup.cfg' -o -name '.mypy.ini' \) | head -3

# Discover the project's linting configuration
find . -type f \( -name '.pylintrc' -o -name 'pylintrc' -o -name '.flake8' \) | head -3

# Search for direct channel.send() calls in interaction callback contexts
grep -r 'channel\.send\|ctx\.send' --include='*.py' | grep -i 'interaction\|callback\|button\|modal' || echo "No direct channel.send() in interaction contexts found"

# Verify interaction.response usage in command handlers
grep -r 'interaction\.response\.' --include='*.py' | wc -l

# Verify ephemeral messaging usage in error handling
grep -r 'ephemeral.*True' --include='*.py' | wc -l
```

**Accept when:**
- All command handlers and UI component callbacks use `interaction.response.*` methods for initial responses, with no direct `channel.send()` calls in interaction contexts.
- Ephemeral messaging (`ephemeral=True`) is used for all error responses and validation failures in command handlers.
- Integration tests pass for interaction response patterns across all command handler modules, validating token-based response delivery.
- Static analysis confirms all `interaction.response` API calls match the locked discord.py library version's type signatures.
- No violations of the interaction response pattern are detected by linting rules flagging direct channel messaging in interaction callback contexts.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review acceptance. Violations must be caught during static analysis and integration testing before merge.
</enforcement>