# Standardize Next.js Configuration as Public API Contract: Configuration Changes That

These rules are ALWAYS ACTIVE for Next.js configuration files (next.config.ts) and any changes that affect public API behavior such as routes, headers, redirects, middleware, and build optimization settings.

### Rules

- **R-NEXTJS-CONFIG-001** MUST: Configuration changes that affect public API behavior (routes, headers, redirects) MUST be reviewed as contract modifications.
- **R-NEXTJS-CONFIG-002** MUST: Place next.config.ts at the root of each Next.js application directory, not at monorepo root.
- **R-NEXTJS-CONFIG-003** MUST: Import Next.js types (NextConfig) to ensure type safety: `import type { NextConfig } from 'next'`.
- **R-NEXTJS-CONFIG-004** MUST: Use environment variables for secrets and environment-specific values; never hardcode them in configuration.
- **R-NEXTJS-CONFIG-005** SHOULD: Document any non-standard configuration options with comments explaining their purpose and impact.
- **R-NEXTJS-CONFIG-006** SHOULD: Keep configuration files simple and extract complex logic to separate modules with tests.
- **R-NEXTJS-CONFIG-007** SHOULD: Test configuration changes in development environment before deploying to production.

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
- Configuration changes affecting public API behavior have been reviewed
- No secrets or hardcoded environment-specific values are present in the configuration

<enforcement>
Claude Code MUST NOT skip or defer verification. Build fails if next.config.ts is missing or contains syntax errors. Next.js startup fails if configuration schema is invalid. CI/CD pipeline blocks deployment if configuration validation fails.
</enforcement>