# Discord.py Interaction Response API for Command and UI Component Messaging: Implementations Discover Exact Resolved Version Discord

These rules are ALWAYS ACTIVE for all Discord slash command handlers, UI component callbacks (buttons, dropdowns, select menus, modals), command routers, and any code path that receives a discord.Interaction object and must respond to the user within the 3-second interaction token window.

### Rules

- **R-DISCORD-001** MUST: Implementations MUST discover the exact resolved version of the Discord library from the project's dependency lock file before using interaction response APIs.
- **R-DISCORD-002** MUST: All command handlers and UI component callbacks MUST use interaction.response.* methods for initial responses, with no direct channel.send() calls in interaction contexts.
- **R-DISCORD-003** MUST: Ephemeral messaging MUST be used for all error responses and validation failures in command handlers.
- **R-DISCORD-004** MUST: Interaction response calls MUST be wrapped in try-except blocks to handle token expiration and API errors gracefully, providing fallback error messages through ephemeral responses when possible.
- **R-DISCORD-005** SHOULD: Implement async callback methods for all UI components (buttons, dropdowns, modals) that receive interaction objects, and use interaction.response.edit_message to update the existing message state rather than creating new messages.
- **R-DISCORD-006** SHOULD: For operations that cannot complete within 3 seconds, implement deferred response pattern using interaction.response.defer() followed by interaction.followup.send with results.
- **R-DISCORD-007** SHOULD: For multi-step workflows involving modals, store intermediate state in the view or modal instance to maintain context across interaction callbacks.

### Verify

```bash
# 1. Discover the project's dependency lock file and identify the exact resolved Discord library version
LOCK_FILE=$(find . -maxdepth 2 -type f \( -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'requirements.lock' -o -name 'uv.lock' \) | head -1)
if [ -z "$LOCK_FILE" ]; then echo "ERROR: No lock file found"; exit 1; fi
echo "Lock file: $LOCK_FILE"

# 2. Extract the exact resolved Discord library version
DISCORD_VERSION=$(grep -A 5 'name = "discord' "$LOCK_FILE" | grep 'version' | head -1 | sed 's/.*version = "\([^"]*\)".*/\1/')
echo "Resolved Discord version: $DISCORD_VERSION"

# 3. Discover the project's test suite location and execute integration tests
TEST_DIR=$(find . -maxdepth 3 -type d \( -name 'tests' -o -name 'test' \) | head -1)
if [ -n "$TEST_DIR" ]; then
  echo "Running integration tests from: $TEST_DIR"
  # Execute tests (command varies by test runner)
  if [ -f "$TEST_DIR/../pytest.ini" ] || [ -f "pytest.ini" ]; then
    pytest "$TEST_DIR" -v -k interaction
  fi
fi

# 4. Discover the project's static analysis configuration and run type checking
if [ -f "pyproject.toml" ]; then
  echo "Running mypy type checking..."
  mypy . --ignore-missing-imports 2>/dev/null || true
fi

# 5. Verify no direct channel.send() calls in interaction callback contexts
echo "Scanning for direct channel.send() in interaction contexts..."
grep -r 'channel\.send' . --include='*.py' | grep -E '(command|interaction|callback|handler)' || echo "No direct channel.send() found in interaction contexts"

# 6. Verify interaction.response.* usage in command handlers
echo "Verifying interaction.response.* usage..."
grep -r 'interaction\.response\.' . --include='*.py' | wc -l

# 7. Verify ephemeral messaging in error handlers
echo "Verifying ephemeral messaging in error responses..."
grep -r 'ephemeral.*=.*True' . --include='*.py' | wc -l
```

**Accept when:**
- The exact resolved Discord library version is discovered from the lock file and documented
- All command handlers and UI component callbacks use interaction.response.* methods for initial responses, with no direct channel.send() calls in interaction contexts
- Ephemeral messaging is used for all error responses and validation failures in command handlers
- Integration tests pass for interaction response patterns across all command handler modules, validating token-based response delivery
- Static analysis confirms all interaction.response API calls match the locked library version's type signatures
- Try-except blocks wrap all interaction response calls to handle token expiration gracefully
- Deferred response patterns are implemented for operations exceeding the 3-second window

<enforcement>
Claude Code MUST NOT skip or defer verification. The lock-file version discovery (steps 1-2) is MANDATORY before any interaction response API usage. Integration tests and static analysis (steps 3-4) MUST execute successfully. All scanning commands (steps 5-7) MUST complete without violations.
</enforcement>