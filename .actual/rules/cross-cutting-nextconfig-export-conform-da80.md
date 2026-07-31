# Standardize Next.js Configuration as Public API Contract: Nextconfig Export Conform

These rules are ALWAYS ACTIVE for all Next.js configuration files (next.config.ts) in the monorepo, particularly in applications like monorepo/tlt/nextjs_app/dashboard/.

### Rules

- **R-NEXTCONFIG-001** MUST: The nextConfig export MUST conform to the Next.js configuration schema as defined by the framework version in use.
- **R-NEXTCONFIG-002** MUST: Place next.config.ts at the root of each Next.js application directory, not at monorepo root.
- **R-NEXTCONFIG-003** MUST: Import Next.js types (NextConfig) to ensure type safety: `import type { NextConfig } from 'next'`.
- **R-NEXTCONFIG-004** MUST: Use environment variables for secrets and environment-specific values; never hardcode them in configuration.
- **R-NEXTCONFIG-005** SHOULD: Document any non-standard configuration options with comments explaining their purpose and impact.
- **R-NEXTCONFIG-006** SHOULD: Keep configuration files simple and extract complex logic to separate modules with tests.

### Verify

```bash
# Verify configuration file exists
test -f monorepo/tlt/nextjs_app/dashboard/next.config.ts && echo 'Configuration file exists'

# Verify nextConfig export is present
grep -q 'nextConfig' monorepo/tlt/nextjs_app/dashboard/next.config.ts && echo 'nextConfig export found'

# Verify TypeScript configuration is valid
npx tsc --noEmit monorepo/tlt/nextjs_app/dashboard/next.config.ts && echo 'TypeScript configuration valid'
```

**Accept when:**
- next.config.ts exists in the Next.js application root directory
- Configuration file exports a valid nextConfig object conforming to Next.js schema
- TypeScript compilation succeeds without errors for the configuration file
- No hardcoded secrets or environment-specific values are present in the configuration
- Non-standard configuration options include explanatory comments

<enforcement>
Claude Code MUST NOT skip or defer verification. Build fails if next.config.ts is missing or contains syntax errors. Next.js startup fails if configuration schema is invalid. CI/CD pipeline blocks deployment if configuration validation fails.
</enforcement>