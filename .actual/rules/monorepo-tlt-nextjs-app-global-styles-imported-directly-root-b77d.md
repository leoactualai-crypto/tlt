# Adopt Next.js Framework with App Router for Dashboard Application: Global Styles Imported Directly Root Layout

These rules are ALWAYS ACTIVE for all React components, pages, and layout files within the dashboard application using Next.js App Router.

### Rules

- **R-NEXTJS-APP-001** MAY: Global styles MAY be imported directly in root layout components following Next.js CSS handling conventions.
- **R-NEXTJS-APP-002** MUST: Root layout components in the app directory define shared UI structure and metadata for all routes; changes to root layouts affect the entire application and require testing across all routes.
- **R-NEXTJS-APP-003** MUST: Server Components are the default in App Router; client-side interactivity requires explicit client component boundaries marked with `'use client'` directive.
- **R-NEXTJS-APP-004** SHOULD: Understand rendering boundary implications before choosing component type (server vs client).
- **R-NEXTJS-APP-005** MUST: Framework configuration files control build behavior, routing rules, and optimization settings; changes must be tested across development and production builds.

### Verify

```bash
# Discover the project's dependency manifest and lock file; verify the framework dependency is declared and resolved
grep -E '(next|dependencies)' package.json
ls -la package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -1

# Discover the project's build and development scripts; execute them to verify the application builds and runs successfully
grep -E '(build|dev)' package.json
npm run build
npm run dev &
sleep 5
curl http://localhost:3000 || echo "Dev server not responding"

# Discover the app directory structure; verify root layout components exist and follow App Router conventions
find . -path './app/layout.*' -type f
grep -l 'export default' ./app/layout.* 2>/dev/null
```

**Accept when:**
- The framework dependency (Next.js) is present in the dependency manifest with a resolved version in the lock file
- The application builds without errors and serves successfully in development mode
- App directory structure contains valid layout components (app/layout.tsx or app/layout.jsx) that render without runtime errors
- Root layout exports a default component and defines shared metadata/structure
- Global styles are imported at the root layout level following Next.js conventions

<enforcement>
Claude Code MUST NOT skip or defer verification. All verification commands MUST be executed before accepting changes to Next.js App Router configuration or root layout components. Build failures or missing framework dependencies block deployment.
</enforcement>