# Discord.py Interaction Response API for Command and UI Component Messaging: Command Handlers Use Interaction Response Send

These rules are ALWAYS ACTIVE for all Discord slash command handlers, UI component callbacks (buttons, dropdowns, select menus, modals), command routers, and any code path that receives a `discord.Interaction` object and must respond to the user within the Discord adapter layer.

### Rules

- **R-DISCORD-INTERACTION-001** SHOULD: Command handlers SHOULD use `interaction.response.send_message()` for initial responses to user interactions.
- **R-DISCORD-INTERACTION-002** SHOULD: Follow-up messages SHOULD use `interaction.followup.send()` when additional messages are required after the initial response.
- **R-DISCORD-INTERACTION-003** MUST: Do not use direct `channel.send()` calls in command handler or UI component callback contexts that receive interaction objects.
- **R-DISCORD-INTERACTION-004** SHOULD: Ephemeral messaging SHOULD be used for all error responses and validation failures in command handlers.
- **R-DISCORD-INTERACTION-005** SHOULD: Long-running command operations SHOULD implement deferred response pattern using `interaction.response.defer()` for operations that cannot complete within 3 seconds, followed by `interaction.followup.send()` with results.
- **R-DISCORD-INTERACTION-006** MUST: Wrap interaction response calls in try-except blocks to handle token expiration and API errors gracefully, providing fallback error messages through ephemeral responses when possible.
- **R-DISCORD-INTERACTION-007** SHOULD: Implement async callback methods for all UI components (buttons, dropdowns, modals) that receive interaction objects, and use `interaction.response.edit_message()` to update the existing message state rather than creating new messages.
- **R-DISCORD-INTERACTION-008** SHOULD: For multi-step workflows involving modals, store intermediate state in the view or modal instance to maintain context across interaction callbacks.

### Verify

```bash
# Discover the project's test suite location and execute integration tests
# that validate interaction response patterns in command handlers
find . -type f -name "*test*.py" -o -name "test_*" | head -5

# Discover the project's static analysis configuration and run type checking
# to verify all interaction.response API calls match the locked library version
find . -type f \( -name "pyproject.toml" -o -name "setup.cfg" -o -name "mypy.ini" \) | head -3

# Discover the project's linting configuration and verify that direct channel
# messaging APIs are flagged when used in command handler or UI callback contexts
find . -type f \( -name ".pylintrc" -o -name "pylintrc" -o -name ".flake8" \) | head -3

# Search for direct channel.send() calls in command handler files
grep -r "channel\.send(" --include="*.py" | grep -E "(command|handler|callback)" || echo "No direct channel.send() found in handler contexts"

# Verify interaction.response.send_message usage in command handlers
grep -r "interaction\.response\.send_message" --include="*.py" | wc -l

# Verify interaction.followup.send usage for follow-up messages
grep -r "interaction\.followup\.send" --include="*.py" | wc -l
```

**Accept when:**
- All command handlers and UI component callbacks use `interaction.response.*` methods for initial responses, with no direct `channel.send()` calls in interaction contexts
- Ephemeral messaging is used for all error responses and validation failures in command handlers
- Integration tests pass for interaction response patterns across all command handler modules, validating token-based response delivery
- Static analysis type checking confirms all `interaction.response` API calls match the locked Discord.py library version's type signatures
- Linting rules flag any direct channel messaging in command handler or UI callback contexts
- All long-running operations implement deferred response patterns with proper error handling

<enforcement>
Claude Code MUST NOT skip or defer verification. All interaction response patterns MUST be validated against the locked library version before implementation. Code review MUST reject pull requests that use direct channel messaging in command handlers or UI callbacks. Integration tests MUST pass for all interaction response patterns.
</enforcement>