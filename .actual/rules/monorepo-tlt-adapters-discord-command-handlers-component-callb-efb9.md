# Discord.py Interaction Response API for Command and UI Component Messaging: Discord Command Handlers Component Callbacks Use

These rules are ALWAYS ACTIVE for all Discord slash command handlers, UI component callbacks (buttons, dropdowns, select menus, modals), command routers, and any code path that receives a `discord.Interaction` object and must respond to the user within the Discord adapter layer.

### Rules

- **R-DISCORD-INTERACTION-001** MUST: All Discord command handlers and UI component callbacks MUST use the interaction response API (`interaction.response.send_message`, `interaction.response.edit_message`, `interaction.response.send_modal`) for initial responses to user interactions.
- **R-DISCORD-INTERACTION-002** MUST: Implement async callback methods for all UI components (buttons, dropdowns, modals) that receive interaction objects, and use `interaction.response.edit_message` to update the existing message state rather than creating new messages.
- **R-DISCORD-INTERACTION-003** MUST: Wrap interaction response calls in try-except blocks to handle token expiration and API errors gracefully, providing fallback error messages through ephemeral responses when possible.
- **R-DISCORD-INTERACTION-004** MUST: For operations that cannot complete within 3 seconds, implement deferred response pattern using `interaction.response.defer()` followed by `interaction.followup.send` with results.
- **R-DISCORD-INTERACTION-005** SHOULD: Use ephemeral messaging for all error responses and validation failures in command handlers to provide user-specific feedback without channel pollution.
- **R-DISCORD-INTERACTION-006** SHOULD: For multi-step workflows involving modals, store intermediate state in the view or modal instance to maintain context across interaction callbacks.

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

# Search for direct channel.send() calls in command handler and callback contexts
grep -r 'channel\.send\|ctx\.send' --include='*.py' | grep -E '(command|handler|callback|interaction)' || echo 'No direct channel.send() calls found in interaction contexts'

# Verify interaction.response API usage in command handlers
grep -r 'interaction\.response\.' --include='*.py' | wc -l
```

**Accept when:**
- All command handlers and UI component callbacks use `interaction.response.*` methods for initial responses, with no direct `channel.send()` calls in interaction contexts
- Ephemeral messaging is used for all error responses and validation failures in command handlers
- Integration tests pass for interaction response patterns across all command handler modules, validating token-based response delivery
- Static analysis confirms all `interaction.response` API calls match the locked discord.py library version's type signatures
- No violations of the 3-second interaction token window are detected in integration test runs

<enforcement>
Claude Code MUST NOT skip or defer verification. All command handlers and UI component callbacks MUST be audited for compliance with R-DISCORD-INTERACTION-001 through R-DISCORD-INTERACTION-006 before code acceptance. Integration tests MUST pass and static analysis MUST confirm API compatibility with the locked library version.
</enforcement>