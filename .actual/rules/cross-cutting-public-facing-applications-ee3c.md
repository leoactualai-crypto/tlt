# Adopt Next.js as Standard Framework for Public API Configuration: Public Facing Applications

These rules are ALWAYS ACTIVE for all public-facing applications and API configuration files in Next.js projects within the monorepo.

### Rules

- **R-NEXTJS-001** MUST: Public-facing applications MUST use Next.js as the framework for API configuration and routing.
- **R-NEXTJS-002** MUST: All Next.js applications MUST contain a `next.config.ts` file at the root of each Next.js application directory.
- **R-NEXTJS-003** MUST: The `nextConfig` export MUST be present and properly typed in all Next.js configuration files using TypeScript imports (e.g., `import type { NextConfig } from 'next'`).
- **R-NEXTJS-004** SHOULD: Isolate Next.js-specific code in configuration and routing layers to maintain clear separation between business logic and framework code.
- **R-NEXTJS-005** SHOULD: Document any custom configuration extensions or plugins in comments within the configuration file.
- **R-NEXTJS-006** SHOULD: Consider creating shared configuration utilities in a common package for settings used across multiple Next.js applications.

### Verify

```bash
# Find all Next.js configuration files with nextConfig export
find monorepo -name 'next.config.ts' -type f | xargs grep -l 'nextConfig'

# Verify Next.js is detected as core library in public API contexts
grep -r "libs.core.detected.*next" monorepo/

# Find Next.js configuration files in application directories
find monorepo -path '*/nextjs_app/*/next.config.ts' -type f

# Verify TypeScript compilation of Next.js configuration files
npx tsc --noEmit monorepo/**/next.config.ts
```

**Accept when:**
- All Next.js applications contain a `next.config.ts` file with TypeScript typing
- The `nextConfig` export is present and properly typed in all Next.js configuration files
- Detection pipeline identifies Next.js as a core library (`libs.core.detected='next'`) in public API contexts
- CI/CD pipeline successfully compiles all Next.js configuration files without TypeScript errors

<enforcement>
Claude Code MUST NOT skip or defer verification. All Next.js applications must comply with these rules before code review approval.
</enforcement>