# Adopt Next.js as Standard Framework for Public API Configuration: Applications Extend Base

These rules are ALWAYS ACTIVE for all Next.js applications in the monorepo that expose public or external APIs, particularly those with explicit next.config.ts configuration files.

### Rules

- **R-NEXTJS-001** MAY: Applications MAY extend the base Next.js configuration with custom plugins and middleware.
- **R-NEXTJS-002** MUST: All Next.js applications contain a next.config.ts file with TypeScript typing.
- **R-NEXTJS-003** MUST: The nextConfig export is present and properly typed in all Next.js configuration files.
- **R-NEXTJS-004** MUST: Place all Next.js configuration in next.config.ts files at the root of each Next.js application directory.
- **R-NEXTJS-005** SHOULD: Use TypeScript imports for Next.js types (e.g., import type { NextConfig } from 'next') to ensure type safety.
- **R-NEXTJS-006** SHOULD: Document any custom configuration extensions or plugins in comments within the configuration file.
- **R-NEXTJS-007** SHOULD: Consider creating shared configuration utilities in a common package for settings used across multiple Next.js applications.

### Verify

```bash
# Find all next.config.ts files and verify nextConfig export
find monorepo -name 'next.config.ts' -type f | xargs grep -l 'nextConfig'

# Verify Next.js is detected as core library in public API contexts
grep -r "libs.core.detected.*next" monorepo/

# Find Next.js applications in standard paths
find monorepo -path '*/nextjs_app/*/next.config.ts' -type f

# Verify TypeScript compilation of Next.js configuration files
npm run build --workspaces
```

**Accept when:**
- All Next.js applications contain a next.config.ts file with TypeScript typing
- The nextConfig export is present and properly typed in all Next.js configuration files
- Detection pipeline identifies Next.js as a core library (libs.core.detected='next') in public API contexts
- CI/CD pipeline successfully compiles all Next.js configuration files without TypeScript errors

<enforcement>
Claude Code MUST NOT skip or defer verification. All Next.js applications must comply with configuration standards before merge. Violations result in CI build failures and code review feedback requesting adherence to these standards.
</enforcement>