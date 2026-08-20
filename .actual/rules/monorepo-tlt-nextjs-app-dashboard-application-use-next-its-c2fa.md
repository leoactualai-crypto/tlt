# Adopt Next.js Framework with App Router for Dashboard Application: Dashboard Application Use Next Its React

These rules are ALWAYS ACTIVE for all React components, pages, routing logic, layouts, and configuration files within the dashboard application.

### Rules

- **R-NEXTJS-001** MUST: The dashboard application MUST use Next.js as its React framework, establishing Next.js as the architectural foundation for routing, rendering, and optimization.
- **R-NEXTJS-002** MUST: All React components and pages within the dashboard application MUST follow App Router conventions (app directory structure).
- **R-NEXTJS-003** MUST: Root layout components in the app directory MUST define shared UI structure and metadata for all routes.
- **R-NEXTJS-004** SHOULD: Server Components SHOULD be the default; client-side interactivity SHOULD require explicit client component boundaries with `'use client'` directive.
- **R-NEXTJS-005** SHOULD: Framework configuration files SHOULD be tested across development and production builds before deployment.
- **R-NEXTJS-006** MAY: Alternative frameworks for new applications MAY be considered only after comparison analysis and team consensus.

### Verify

```bash
# Discover the project's dependency manifest and lock file; verify the framework dependency is declared and resolved
grep -E '"next"|next:' package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -5

# Discover the project's build and development scripts; execute them to verify the application builds and runs successfully
cat package.json | grep -A 5 '"scripts"'

# Discover the app directory structure; verify root layout components exist and follow App Router conventions
find . -path './app/layout.tsx' -o -path './app/layout.jsx' 2>/dev/null

# Verify the application builds without errors
npm run build 2>&1 | tail -20
```

**Accept when:**
- The framework dependency is present in the dependency manifest with a resolved version in the lock file
- The application builds without errors and serves successfully in development mode
- App directory structure contains valid layout components that render without runtime errors
- Root layout components exist at `app/layout.tsx` or `app/layout.jsx`
- No Pages Router (`pages/` directory) is used for new routes

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for dashboard application code. Build failures from missing framework dependencies or incorrect App Router usage block deployment.
</enforcement>