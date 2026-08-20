# Adopt Next.js Framework with App Router for Dashboard Application: Implementations Use App Router Architecture Directory

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The dashboard application requires a React-based framework with integrated routing, server-side rendering capabilities, and built-in optimization features
- Next.js provides a cohesive solution combining React rendering strategies, file-system-based routing, and performance optimizations without requiring separate library integrations
- The App Router architecture (app directory structure) was chosen over the legacy Pages Router, indicating alignment with React Server Components and modern Next.js patterns
- TypeScript configuration and font optimization APIs demonstrate adoption of Next.js ecosystem tooling for type safety and performance

## Problem Statement

The dashboard application needs a React framework that provides integrated routing, multiple rendering strategies (SSR, SSG, CSR), automatic code splitting, and built-in optimization features while maintaining developer productivity and type safety. Assembling these capabilities from separate libraries increases configuration complexity and integration overhead.

## Decision

1. MUST: Implementations MUST use the App Router architecture with app directory structure for layouts and pages

## Policy Block

- MUST Implementations MUST use the App Router architecture with app directory structure for layouts and pages

In scope:
- All React components and pages within the dashboard application
- Routing and navigation logic for the dashboard
- Server-side rendering and static generation requirements
- Application-level layouts and metadata configuration

Out of scope:
- Non-React applications or services in the monorepo
- Standalone React applications that do not require Next.js features
- Backend services or APIs that are framework-agnostic
- Build tooling or infrastructure code outside the dashboard application

## Rationale

- Next.js provides an integrated solution for React application concerns (routing, rendering, optimization) that would otherwise require assembling and configuring multiple separate libraries
- The App Router architecture aligns with React Server Components and modern React patterns, providing a forward-compatible foundation
- Built-in optimization features (font loading, image optimization, code splitting) reduce manual performance engineering effort
- The evidence shows adoption across configuration and layout files, indicating deliberate framework integration rather than experimental usage

## Consequences

Positive:
- Integrated routing, rendering, and optimization reduce configuration complexity and library integration overhead
- Built-in TypeScript support and type-safe configuration improve developer experience and catch errors early
- Automatic code splitting and optimization features improve application performance without manual intervention
- Strong ecosystem and documentation support accelerate feature development

Negative:
- Framework coupling introduces dependency on Next.js release cycle and breaking changes across major versions
- App Router architecture requires understanding Next.js-specific conventions and rendering boundaries (server vs client components)
- Migration to alternative frameworks becomes more costly due to Next.js-specific APIs and patterns
- Framework abstractions may obscure underlying React and build tool behavior, increasing debugging complexity

## Alternatives

- Use Create React App or Vite with separate routing library (React Router) (rejected)
  Rejected because: Requires manual integration of routing, SSR capabilities, and optimization features; increases configuration surface area and maintenance burden
  When valid: Valid for simple single-page applications without server-rendering requirements or when framework independence is a hard constraint
- Use Next.js Pages Router instead of App Router (rejected)
  Rejected because: Evidence shows App Router adoption (app/layout.tsx structure); Pages Router lacks React Server Components support and represents legacy architecture
  When valid: Valid for existing Next.js applications not yet migrated or when App Router features are not required
- Use alternative React meta-framework (Remix, Gatsby) (rejected)
  Rejected because: Evidence explicitly shows Next.js adoption; alternative frameworks have different rendering models and ecosystem maturity
  When valid: Valid when specific framework features align better with application requirements (e.g., Remix for form-heavy applications, Gatsby for content-focused sites)

## Risks

- Next.js major version upgrades may introduce breaking changes requiring significant refactoring, particularly around App Router conventions
  Mitigation: Pin major version in dependency manifest; allocate dedicated upgrade cycles for major version migrations; maintain test coverage for framework integration points
  Owner: engineering team
- Developers unfamiliar with Next.js conventions may misuse server/client component boundaries, causing runtime errors or performance issues
  Mitigation: Establish team training on Next.js architecture; document server/client component patterns; implement linting rules to catch common mistakes
  Owner: engineering team
- Framework abstractions may hide performance bottlenecks or make debugging complex rendering issues more difficult
  Mitigation: Use Next.js built-in profiling and debugging tools; maintain understanding of underlying React and build tool behavior; document known framework-specific debugging approaches
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Root layout components in the app directory define shared UI structure and metadata for all routes; changes to root layouts affect the entire application
- Server Components are the default in App Router; client-side interactivity requires explicit client component boundaries; understand the rendering boundary implications before choosing component type
- Framework configuration files control build behavior, routing rules, and optimization settings; changes should be tested across development and production builds

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and lock file; verify the framework dependency is declared and resolved
- Discover the project's build and development scripts; execute them to verify the application builds and runs successfully
- Discover the app directory structure; verify root layout components exist and follow App Router conventions

Accept when:
- The framework dependency is present in the dependency manifest with a resolved version in the lock file
- The application builds without errors and serves successfully in development mode
- App directory structure contains valid layout components that render without runtime errors

## Enforcement

- Verified by: Dependency manifest inspection confirms framework presence
- Verified by: Build process execution verifies framework integration
- Verified by: Code review validates App Router conventions and API usage patterns
- Violation handling: Build failures from missing framework dependencies block deployment
- Violation handling: Code review identifies deviations from App Router conventions and requests corrections
- Violation handling: Runtime errors from incorrect API usage are surfaced in development and testing environments
- Exception process: Exceptions to framework usage require architectural review and documentation of alternative approach
- Exception process: Temporary deviations during migration or experimentation must be tracked and time-boxed
- Exception process: Alternative frameworks for new applications require comparison analysis and team consensus