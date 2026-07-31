# Adopt Next.js as Standard Framework for Public API Configuration: Nextconfig Contract Exported

These rules are ALWAYS ACTIVE for all Next.js applications and configuration files in the monorepo, particularly those exposing public API contracts through next.config.ts.

### Rules

- **R-NEXTCONFIG-001** MUST: The nextConfig contract MUST be exported as the public API configuration interface from all next.config.ts files.
- **R-NEXTCONFIG-002** MUST: All Next.js configuration files MUST use TypeScript typing with proper imports from the 'next' package (e.g., `import type { NextConfig } from 'next'`).
- **R-NEXTCONFIG-003** MUST: All Next.js applications MUST contain a next.config.ts file at the root of the application directory with a properly typed and exported nextConfig object.
- **R-NEXTCONFIG-004** SHOULD: Custom configuration extensions or plugins SHOULD be documented in comments within the configuration file.
- **R-NEXTCONFIG-005** SHOULD: Shared configuration utilities for settings used across multiple Next.js applications SHOULD be created in a common package to promote reusability.

### Verify

```bash
# Find all next.config.ts files and verify nextConfig export
find monorepo -name 'next.config.ts' -type f | xargs grep -l 'nextConfig'

# Verify Next.js is detected as core library in public API contexts
grep -r "libs.core.detected.*next" monorepo/

# Find Next.js configuration files in nextjs_app directories
find monorepo -path '*/nextjs_app/*/next.config.ts' -type f

# Verify TypeScript typing in configuration files
find monorepo -name 'next.config.ts' -type f | xargs grep -l 'import type.*NextConfig.*from.*next'
```

**Accept when:**
- All Next.js applications contain a next.config.ts file with TypeScript typing
- The nextConfig export is present and properly typed in all Next.js configuration files
- Detection pipeline identifies Next.js as a core library (libs.core.detected='next') in public API contexts
- TypeScript compilation of Next.js configuration files succeeds without errors

<enforcement>
Claude Code MUST NOT skip or defer verification. All Next.js applications MUST comply with these configuration standards before code review approval. CI/CD pipeline checks MUST enforce TypeScript compilation of next.config.ts files.
</enforcement>