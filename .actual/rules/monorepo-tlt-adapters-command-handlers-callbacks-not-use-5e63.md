# Discord.py Interaction Response API for Command and UI Component Messaging: Command Handlers Callbacks Not Use Direct

These rules are ALWAYS ACTIVE for all Discord slash command handlers, UI component callbacks (buttons, dropdowns, select menus, modals), command routers, and any code path that receives a `discord.Interaction` object and must respond to the user within the interaction token window.

### Rules

- **R-DISCORD-001** MUST NOT: Command handlers and UI callbacks MUST NOT use direct channel messaging APIs (`channel.send`) for initial interaction responses as this bypasses the interaction token contract and fails to provide ephemeral messaging capabilities.
- **R-DISCORD-002** MUST: All command handlers and UI component callbacks MUST use `interaction.response.*` methods for initial responses to satisfy Discord's 3-second interaction token contract.
- **R-DISCORD-003** MUST: Ephemeral messaging MUST be used for all error responses and validation failures in command handlers to provide user-specific feedback without channel pollution.
- **R-DISCORD-004** SHOULD: Implement async callback methods for all UI components (buttons, dropdowns, modals) that receive interaction objects, and use `interaction.response.edit_message` to update existing message state rather than creating new messages.
- **R-DISCORD-005** SHOULD: Wrap interaction response calls in try-except blocks to handle token expiration and API errors gracefully, providing fallback error messages through ephemeral responses when possible.
- **R-DISCORD-006** SHOULD: For long-running command operations that cannot complete within 3 seconds, implement deferred response pattern using `interaction.response.defer()` followed by `interaction.followup.send` with results.
- **R-DISCORD-007** SHOULD: For multi-step workflows involving modals, store intermediate state in the view or modal instance to maintain context across interaction callbacks, as interaction tokens do not persist state between responses.

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
grep -r 'channel\.send' --include='*.py' | grep -E '(command|handler|callback|interaction)' || echo 'No direct channel.send() calls found in interaction contexts'

# Verify interaction.response usage in command handlers
grep -r 'interaction\.response\.' --include='*.py' | wc -l

# Check for ephemeral messaging patterns in error handling
grep -r 'ephemeral.*True' --include='*.py' | wc -l
```

**Accept when:**
- All command handlers and UI component callbacks use `interaction.response.*` methods for initial responses, with no direct `channel.send()` calls in interaction contexts
- Ephemeral messaging is used for all error responses and validation failures in command handlers
- Integration tests pass for interaction response patterns across all command handler modules, validating token-based response delivery
- Static analysis confirms all `interaction.response` API calls match the locked discord.py library version's type signatures
- No violations of the direct channel messaging prohibition are detected in command handler or UI callback code paths

<enforcement>
Claude Code MUST NOT skip or defer verification. All command handlers and UI callbacks MUST be audited to confirm exclusive use of `interaction.response.*` APIs for initial responses. Integration tests MUST pass before accepting changes. Static analysis MUST confirm API compatibility with the locked library version.
</enforcement>