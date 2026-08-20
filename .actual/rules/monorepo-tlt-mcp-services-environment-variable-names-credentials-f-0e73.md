# Environment Variable Retrieval for API Credentials: Environment Variable Names Credentials Follow Consistent

These rules are ALWAYS ACTIVE for backend services that authenticate with external APIs and require secret credentials for third-party integrations.

### Rules

- **R-ENV-001** SHOULD: Environment variable names for API credentials SHOULD follow a consistent naming convention that identifies the service and credential type.
- **R-ENV-002** MUST: Validate that all required environment variables are present during service initialization, before attempting to use external APIs, to fail fast with clear error messages.
- **R-ENV-003** MUST: Document required environment variables in service README files, deployment guides, and infrastructure-as-code templates to ensure proper configuration across environments.
- **R-ENV-004** MUST: Never hardcode credentials directly in source files.
- **R-ENV-005** SHOULD: Implement a configuration validation module that centralizes environment variable retrieval and validation logic to ensure consistent error handling.
- **R-ENV-006** MUST: Implement logging filters to redact credential values and handle credential retrieval errors without exposing values.

### Verify

```bash
# Discover and execute the project's static analysis or linting configuration
# to detect hardcoded credential patterns in source files
find . -type f -name '.eslintrc*' -o -name 'pylintrc' -o -name '.flake8' | head -1

# Discover and execute the project's test suite to verify that services
# handle missing environment variables gracefully with appropriate error messages
find . -type f \( -name 'package.json' -o -name 'pytest.ini' -o -name 'setup.py' \) | head -1

# Inspect the repository to confirm no files contain credential values
grep -r "OPENAI_API_KEY\|api_key\|secret" --include="*.js" --include="*.py" --include="*.ts" . 2>/dev/null | grep -v node_modules | grep -v '.git' || echo "No hardcoded credentials found"

# Verify credential-related files are properly listed in version control ignore configurations
cat .gitignore 2>/dev/null | grep -E '\.env|secrets|credentials' || echo "Check .gitignore for credential file patterns"
```

**Accept when:**
- Static analysis confirms no hardcoded credentials exist in source files
- Service initialization tests verify that missing required environment variables produce clear error messages before attempting API calls
- Version control ignore configurations prevent credential files from being committed
- Environment variable names follow a consistent naming convention (e.g., `SERVICE_CREDENTIAL_TYPE`)
- All required environment variables are documented in deployment guides

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST scan for credential patterns. CI pipeline MUST fail builds containing hardcoded credential patterns. Code review MUST verify environment variable usage for new external API integrations. No exceptions permitted for production code.
</enforcement>