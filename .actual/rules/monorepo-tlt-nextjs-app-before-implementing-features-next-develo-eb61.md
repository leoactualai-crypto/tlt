# Adopt Next.js Framework with App Router for Dashboard Application: Before Implementing Features Next Developers Discover

These rules are ALWAYS ACTIVE for all React components, pages, routing logic, server-side rendering, static generation, and application-level layouts within the dashboard application using Next.js App Router.

### Rules

- **R-NEXTJS-001** MUST: Before implementing features using Next.js APIs, developers MUST discover the project's dependency lock file, resolve the exact installed version, and verify API compatibility against that version's official documentation.
- **R-NEXTJS-002** MUST: Execute the lock-version grounding process in order: (1) Find the dependency manifest; (2) Identify the build tool; (3) Inspect the lock or resolution artifact for exact resolved version; (4) Look up official documentation for that exact version; (5) Confirm every API, class, or function exists in that version's documentation before using it; (6) Re-run steps 3–5 per dependency at point of use for version-sensitive behavior.
- **R-NEXTJS-003** MUST: Understand that root layout components in the app directory define shared UI structure and metadata for all routes; changes to root layouts affect the entire application.
- **R-NEXTJS-004** MUST: Recognize that Server Components are the default in App Router; client-side interactivity requires explicit client component boundaries; understand rendering boundary implications before choosing component type.
- **R-NEXTJS-005** MUST: Test framework configuration file changes across both development and production builds, as these files control build behavior, routing rules, and optimization settings.

### Verify

```bash
# Discover the project's dependency manifest and lock file; verify the framework dependency is declared and resolved
grep -E '"next"|next:' package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null || echo "Next.js dependency not found"

# Discover the project's build and development scripts; execute them to verify the application builds and runs successfully
npm run build && npm run dev &
sleep 5
curl http://localhost:3000 || echo "Application failed to start"

# Discover the app directory structure; verify root layout components exist and follow App Router conventions
test -f app/layout.tsx || test -f app/layout.jsx || echo "Root layout not found in app directory"
```

**Accept when:**
- The framework dependency is present in the dependency manifest with a resolved version in the lock file
- The application builds without errors and serves successfully in development mode
- App directory structure contains valid layout components that render without runtime errors
- All Next.js APIs used in code are confirmed to exist in the exact resolved version's official documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. Dependency lock file inspection and version-specific API confirmation are mandatory before any Next.js feature implementation.
</enforcement>