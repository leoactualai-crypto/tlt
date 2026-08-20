# Environment Variable Retrieval for API Credentials: Services Use Standard Library Environment Variable

These rules are ALWAYS ACTIVE for all backend services that authenticate with external APIs and require secret credentials for third-party integrations.

### Rules

- **R-ENVVAR-001** MUST: Services MUST use the standard library environment variable access function to retrieve secrets.
- **R-ENVVAR-002** MUST: Validate that all required environment variables are present during service initialization, before attempting to use external APIs, to fail fast with clear error messages.
- **R-ENVVAR-003** MUST: Document required environment variables in service README files, deployment guides, and infrastructure-as-code templates to ensure proper configuration across environments.
- **R-ENVVAR-004** SHOULD: Implement a configuration validation module that centralizes environment variable retrieval and validation logic to ensure consistent error handling.
- **R-ENVVAR-005** MUST: Implement logging filters to redact credential values and handle credential retrieval errors without exposing values.
- **R-ENVVAR-006** MUST NOT: Hardcode credentials directly in source files.
- **R-ENVVAR-007** MUST NOT: Store credentials in configuration files that may be committed to version control.

### Verify

```bash
# Discover and execute the project's static analysis or linting configuration
# to detect hardcoded credential patterns in source files
find . -name '.eslintrc*' -o -name 'pylintrc' -o -name '.flake8' -o -name 'sonar-project.properties' | head -1

# Discover and execute the project's test suite to verify that services
# handle missing environment variables gracefully with appropriate error messages
find . -name 'package.json' -o -name 'pytest.ini' -o -name 'setup.py' -o -name 'Makefile' | grep -E '(package\.json|pytest\.ini|setup\.py|Makefile)' | head -1

# Inspect the repository to confirm no files contain credential values
grep -r 'OPENAI_API_KEY\|api_key\|secret_key' --include='*.js' --include='*.ts' --include='*.py' --include='*.java' . 2>/dev/null | grep -v 'process.env\|os.environ\|System.getenv' | head -20

# Verify credential-related files are properly listed in version control ignore configurations
cat .gitignore 2>/dev/null | grep -E '\.env|\.secrets|credentials'
```

**Accept when:**
- Static analysis confirms no hardcoded credentials exist in source files
- Service initialization tests verify that missing required environment variables produce clear error messages before attempting API calls
- Version control ignore configurations prevent credential files from being committed
- All external API integrations retrieve credentials via standard library environment variable functions
- Required environment variables are documented in service configuration guides

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST block commits containing potential credentials. CI pipeline MUST fail builds that contain hardcoded credential patterns. Code review MUST require revision before merge if credentials are not properly externalized. No exceptions are permitted for production code.
</enforcement>