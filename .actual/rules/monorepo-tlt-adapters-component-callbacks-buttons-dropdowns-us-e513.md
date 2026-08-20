# Discord.py Interaction Response API for Command and UI Component Messaging: Component Callbacks Buttons Dropdowns Use Interaction

These rules are ALWAYS ACTIVE for all Discord slash command handlers, UI component callbacks (buttons, dropdowns, select menus, modals), and any code path that receives a discord.Interaction object and must respond to the user within the 3-second interaction token window.

### Rules

- **R-DISCORD-INTERACTION-001** SHOULD: UI component callbacks (buttons, dropdowns) SHOULD use `interaction.response.edit_message()` to update the existing message rather than creating new messages.
- **R-DISCORD-INTERACTION-002** MUST: All command handlers and UI component callbacks MUST use `interaction.response.*` methods for initial responses, with no direct `channel.send()` calls in interaction contexts.
- **R-DISCORD-INTERACTION-003** MUST: Ephemeral messaging MUST be used for all error responses and validation failures in command handlers.
- **R-DISCORD-INTERACTION-004** MUST: Implement async callback methods for all UI components (buttons, dropdowns, modals) that receive interaction objects.
- **R-DISCORD-INTERACTION-005** MUST: Wrap interaction response calls in try-except blocks to handle token expiration and API errors gracefully, providing fallback error messages through ephemeral responses when possible.
- **R-DISCORD-INTERACTION-006** SHOULD: For multi-step workflows involving modals, SHOULD store intermediate state in the view or modal instance to maintain context across interaction callbacks.
- **R-DISCORD-INTERACTION-007** SHOULD: For operations that cannot complete within 3 seconds, SHOULD implement deferred response pattern using `interaction.response.defer()` followed by `interaction.followup.send()` with results.

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
grep -r 'channel\.send\|ctx\.send' --include='*.py' | grep -E '(command|callback|interaction)' || echo "No direct channel.send() found in interaction contexts"

# Verify interaction.response usage in handlers
grep -r 'interaction\.response\.' --include='*.py' | wc -l

# Check for ephemeral messaging patterns
grep -r 'ephemeral\s*=\s*True' --include='*.py' | wc -l
```

**Accept when:**
- All command handlers and UI component callbacks use `interaction.response.*` methods for initial responses, with no direct `channel.send()` calls in interaction contexts
- Ephemeral messaging is used for all error responses and validation failures in command handlers
- Integration tests pass for interaction response patterns across all command handler modules, validating token-based response delivery
- Static analysis confirms all `interaction.response` API calls match the locked discord.py library version's type signatures
- All UI component callbacks are implemented as async methods receiving interaction objects
- Try-except blocks wrap interaction response calls with graceful error handling

<enforcement>
Claude Code MUST NOT skip or defer verification. All interaction response patterns MUST be validated against the locked discord.py version before code acceptance. Code review MUST reject pull requests using direct channel messaging in command handlers or UI callbacks.
</enforcement>