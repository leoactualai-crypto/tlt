# Adopt Next.js as Standard Framework for Public API Configuration: Next Applications Organized

These rules are ALWAYS ACTIVE for all Next.js applications and configuration files within the monorepo, particularly those organized under dedicated application directories (e.g., nextjs_app/dashboard).

### Rules

- **R-NEXT-001** SHOULD: Next.js applications SHOULD be organized under dedicated application directories (e.g., nextjs_app/dashboard).
- **R-NEXT-002** MUST: All Next.js applications MUST contain a next.config.ts file with TypeScript typing at the root of each Next.js application directory.
- **R-NEXT-003** MUST: The nextConfig export MUST be present and properly typed in all Next.js configuration files.
- **R-NEXT-004** SHOULD: Next.js-specific code SHOULD be isolated in configuration and routing layers with clear separation between business logic and framework code.
- **R-NEXT-005** SHOULD: Shared configuration utilities SHOULD be created in a common package for settings used across multiple Next.js applications.
- **R-NEXT-006** MUST: TypeScript imports for Next.js types (e.g., import type { NextConfig } from 'next') MUST be used to ensure type safety.
- **R-NEXT-007** SHOULD: Custom configuration extensions or plugins SHOULD be documented in comments within the configuration file.

### Verify

```bash
# Find all next.config.ts files and verify nextConfig export
find monorepo -name 'next.config.ts' -type f | xargs grep -l 'nextConfig'

# Verify Next.js is detected as core library in public API contexts
grep -r "libs.core.detected.*next" monorepo/

# Find all Next.js applications following the standard directory pattern
find monorepo -path '*/nextjs_app/*/next.config.ts' -type f

# Verify TypeScript compilation of Next.js configuration files
npx tsc --noEmit monorepo/**/next.config.ts
```

**Accept when:**
- All Next.js applications contain a next.config.ts file with TypeScript typing
- The nextConfig export is present and properly typed in all Next.js configuration files
- Detection pipeline identifies Next.js as a core library (libs.core.detected='next') in public API contexts
- TypeScript compilation succeeds for all Next.js configuration files
- Next.js applications are organized under dedicated application directories

<enforcement>
Claude Code MUST NOT skip or defer verification. CI/CD pipeline checks for TypeScript compilation of Next.js configuration files are mandatory. Code review verification that new Next.js applications follow configuration standards is required. Violations result in CI build failures for Next.js applications missing required configuration files.
</enforcement>