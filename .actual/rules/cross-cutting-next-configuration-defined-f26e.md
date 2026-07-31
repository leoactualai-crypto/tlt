# Adopt Next.js as Standard Framework for Public API Configuration: Next Configuration Defined

These rules are ALWAYS ACTIVE for all Next.js applications and public API configuration files within the monorepo.

### Rules

- **R-NEXT-001** MUST: Next.js configuration MUST be defined in next.config.ts with explicit TypeScript typing.
- **R-NEXT-002** MUST: Export the configuration object as 'nextConfig' to maintain consistent naming across applications.
- **R-NEXT-003** MUST: Use TypeScript imports for Next.js types (e.g., import type { NextConfig } from 'next') to ensure type safety.
- **R-NEXT-004** SHOULD: Document any custom configuration extensions or plugins in comments within the configuration file.
- **R-NEXT-005** SHOULD: Consider creating shared configuration utilities in a common package for settings used across multiple Next.js applications.

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
- All Next.js applications contain a next.config.ts file with TypeScript typing
- The nextConfig export is present and properly typed in all Next.js configuration files
- Detection pipeline identifies Next.js as a core library (libs.core.detected='next') in public API contexts
- TypeScript compilation succeeds for all next.config.ts files
- Custom configuration extensions are documented with inline comments

<enforcement>
Claude Code MUST NOT skip or defer verification. All Next.js applications MUST comply with these configuration standards before code review approval.
</enforcement>