# Discord.py Interaction Response API for Command and UI Component Messaging: Error Responses Validation Failures Use Ephemeral

These rules are ALWAYS ACTIVE for all Discord slash command handlers, UI component callbacks (buttons, dropdowns, select menus, modals), command routers, and any code path that receives a `discord.Interaction` object and must respond to the user within the 3-second interaction token window.

### Rules

- **R-DISCORD-INTERACTION-001** MUST: Error responses and validation failures MUST use ephemeral messaging (ephemeral=True parameter) to provide user-specific feedback without polluting shared channels.
- **R-DISCORD-INTERACTION-002** MUST: All command handlers and UI component callbacks MUST use `interaction.response.*` methods for initial responses, with no direct `channel.send()` calls in interaction contexts.
- **R-DISCORD-INTERACTION-003** MUST: Implement async callback methods for all UI components (buttons, dropdowns, modals) that receive interaction objects, and use `interaction.response.edit_message()` to update the existing message state rather than creating new messages.
- **R-DISCORD-INTERACTION-004** MUST: Wrap interaction response calls in try-except blocks to handle token expiration and API errors gracefully, providing fallback error messages through ephemeral responses when possible.
- **R-DISCORD-INTERACTION-005** SHOULD: For long-running command operations that may exceed the 3-second interaction window, implement deferred response pattern using `interaction.response.defer()` followed by `interaction.followup.send()` with results.
- **R-DISCORD-INTERACTION-006** SHOULD: For multi-step workflows involving modals, store intermediate state in the view or modal instance to maintain context across interaction callbacks, as interaction tokens do not persist state between responses.

### Verify

```bash
# Discover the project's test suite location and execute integration tests
# that validate interaction response patterns in command handlers
find . -type f -name '*test*.py' -o -name 'test_*' | head -5

# Discover the project's static analysis configuration and run type checking
# to verify all interaction.response API calls match the locked library version
find . -type f \( -name 'pyproject.toml' -o -name 'setup.cfg' -o -name '.mypy.ini' \) | head -3

# Discover the project's linting configuration and verify that direct channel
# messaging APIs are flagged when used in command handler or UI callback contexts
find . -type f \( -name '.pylintrc' -o -name 'pylintrc' -o -name '.flake8' \) | head -3

# Search for direct channel.send() calls in interaction callback contexts
grep -r 'channel\.send' --include='*.py' | grep -E '(command|handler|callback|interaction)' || echo "No direct channel.send() found in interaction contexts"

# Verify ephemeral=True usage in error response paths
grep -r 'ephemeral\s*=\s*True' --include='*.py' | wc -l

# Verify interaction.response.* usage patterns
grep -r 'interaction\.response\.' --include='*.py' | wc -l
```

**Accept when:**
- All command handlers and UI component callbacks use `interaction.response.*` methods for initial responses, with no direct `channel.send()` calls in interaction contexts
- Ephemeral messaging is used for all error responses and validation failures in command handlers
- Integration tests pass for interaction response patterns across all command handler modules, validating token-based response delivery
- Static analysis confirms all `interaction.response` API calls match the locked discord.py library version's type signatures
- No violations of the interaction response pattern are detected by linting or code review

<enforcement>
Claude Code MUST NOT skip or defer verification. All interaction response patterns MUST be validated before accepting changes to command handlers or UI component callbacks. Violations of ephemeral messaging requirements for error responses MUST result in code review rejection.
</enforcement>