# Use discord.py Modal UI Pattern for Structured Bot Input: Modal Form Fields Defined Class Attributes

These rules are ALWAYS ACTIVE for all Discord bot command implementations within the adapter layer requiring two or more structured input fields, including event creation, update, or configuration flows where users must provide multiple pieces of information.

### Rules

- **R-MODAL-001** MUST: Modal form fields MUST be defined as class attributes using discord.ui.TextInput with explicit configuration for label, placeholder, required status, and max_length constraints.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
find . -name "pyproject.toml" -o -name "requirements.txt" -o -name "Pipfile" -o -name "poetry.lock" | head -1

# 2. Resolve the installed discord library version from lock artifact
grep -E "discord|discord\.py" $(find . -name "*.lock" -o -name "requirements*.txt" | head -1) | head -1

# 3. Locate modal class definitions in the codebase
find . -type f -name "*.py" -exec grep -l "discord\.ui\.Modal" {} \;

# 4. Verify modal classes inherit from discord.ui.Modal and define fields as discord.ui.TextInput class attributes
grep -A 10 "class.*Modal" $(find . -type f -name "*.py" -exec grep -l "discord\.ui\.Modal" {} \;) | grep -E "(class|TextInput|on_submit)"

# 5. Verify on_submit methods are async and receive discord.Interaction parameter
grep -A 5 "async def on_submit" $(find . -type f -name "*.py" -exec grep -l "discord\.ui\.Modal" {} \;) | grep -E "(async|Interaction)"

# 6. Execute project test suite covering modal submission flows
find . -type f -name "test_*.py" -o -name "*_test.py" | xargs grep -l "modal\|Modal" | head -1
```

**Accept when:**
- All modal classes inherit from discord.ui.Modal with title parameter and define fields as discord.ui.TextInput class attributes
- Modal on_submit methods are async, receive discord.Interaction parameter, and delegate to handler classes rather than implementing business logic inline
- Test coverage verifies modal submission flows handle interaction responses within Discord timeout constraints
- TextInput fields include explicit configuration for label, placeholder, required status, and max_length constraints
- Modal instances are created with required bot instance or context dependencies passed to __init__

<enforcement>
Clause Code MUST NOT skip or defer verification. All new Discord command implementations requiring multi-field input MUST comply with R-MODAL-001 before merge. Code review MUST verify modal pattern usage. Static analysis SHOULD detect direct text parsing in command handlers where modal pattern should apply. Test coverage MUST include modal submission flows with interaction handling verification.
</enforcement>