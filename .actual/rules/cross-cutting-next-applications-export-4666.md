# Standardize Next.js Configuration as Public API Contract: Next Applications Export

These rules are ALWAYS ACTIVE for all Next.js applications exporting configuration objects from next.config.ts or next.config.js files.

### Rules

- **R-NEXTJS-001** MUST: Next.js applications MUST export a configuration object named 'nextConfig' from next.config.ts or next.config.js
- **R-NEXTJS-002** MUST: Place next.config.ts at the root of each Next.js application directory, not at monorepo root
- **R-NEXTJS-003** MUST: Import Next.js types (NextConfig) to ensure type safety: `import type { NextConfig } from 'next'`
- **R-NEXTJS-004** MUST: Use environment variables for secrets and environment-specific values; never hardcode them in configuration
- **R-NEXTJS-005** SHOULD: Document any non-standard configuration options with comments explaining their purpose and impact
- **R-NEXTJS-006** SHOULD: Keep configuration files simple and extract complex logic to separate modules with tests

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

<enforcement>
Claude Code MUST NOT skip or defer verification. Build fails if next.config.ts is missing or contains syntax errors. Next.js startup fails if configuration schema is invalid. CI/CD pipeline blocks deployment if configuration validation fails.
</enforcement>