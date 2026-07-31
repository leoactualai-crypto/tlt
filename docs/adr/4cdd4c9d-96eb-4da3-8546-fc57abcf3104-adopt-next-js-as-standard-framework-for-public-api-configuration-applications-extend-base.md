# Adopt Next.js as Standard Framework for Public API Configuration: Applications Extend Base

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Context

- The monorepo contains a Next.js application at monorepo/tlt/nextjs_app/dashboard with explicit Next.js configuration
- The next.config.ts file establishes a public API contract (nextConfig) for framework configuration
- Next.js provides integrated routing, API routes, and server-side rendering capabilities suitable for public-facing applications
- The detection of libs.core.detected='next' indicates Next.js is recognized as a core library dependency in the public API layer

## Problem Statement

The codebase requires a standardized approach to configuring and exposing public/external APIs with consistent framework conventions, type safety, and configuration contracts to ensure maintainability and predictable behavior across the application surface.

## Decision

1. MAY: Applications MAY extend the base Next.js configuration with custom plugins and middleware

## Policy Block

- MAY Applications MAY extend the base Next.js configuration with custom plugins and middleware

## Rationale

- The evidence shows Next.js is explicitly detected as a core library (libs.core.detected='next') in the public API layer, indicating intentional framework selection
- The presence of next.config.ts with a nextConfig contract demonstrates a formalized configuration approach for public API surfaces
- Next.js provides integrated capabilities for API routes, server-side rendering, and static generation that align with public/external API requirements
- The pattern appears in the dashboard application path, suggesting Next.js is the chosen framework for user-facing and external API endpoints

## Consequences

Positive:
- Standardized framework reduces cognitive overhead and improves developer productivity across public API development
- TypeScript configuration provides compile-time validation and IDE support for configuration contracts
- Next.js built-in features (API routes, middleware, edge functions) reduce custom infrastructure code
- Consistent configuration patterns improve maintainability and onboarding for new team members

Negative:
- Framework lock-in to Next.js ecosystem may limit flexibility for alternative API patterns
- Next.js version upgrades may require coordinated configuration migrations across applications
- Learning curve for developers unfamiliar with Next.js conventions and configuration options
- Single framework dependency creates a potential single point of failure for all public APIs

## Alternatives

- Use Express.js or Fastify for API routing with separate frontend framework (rejected)
  Rejected because: Requires maintaining separate frontend and backend frameworks, increasing complexity and reducing integration benefits that Next.js provides through unified routing and rendering
  When valid: When API-only services are needed without any frontend rendering requirements
- Adopt framework-agnostic API gateway pattern with multiple backend frameworks (rejected)
  Rejected because: Increases architectural complexity and operational overhead without clear benefits given the current monorepo structure and dashboard application requirements
  When valid: When supporting polyglot services or migrating from legacy systems with diverse technology stacks
- Use Remix or SvelteKit as alternative full-stack frameworks (rejected)
  Rejected because: Next.js is already established in the codebase with existing configuration contracts; migration would require significant refactoring without demonstrated benefits
  When valid: When starting a new project without existing Next.js dependencies or when specific framework features are required

## Risks

- Next.js breaking changes in major version updates could require extensive configuration refactoring across all applications
  Mitigation: Pin Next.js versions in package.json, establish testing procedures for version upgrades, and maintain migration documentation for configuration changes
  Owner: engineering team
- Over-reliance on Next.js-specific patterns may create portability challenges if framework migration becomes necessary
  Mitigation: Isolate Next.js-specific code in configuration and routing layers, maintain clear separation between business logic and framework code
  Owner: engineering team
- Configuration complexity may grow over time as applications add custom plugins and middleware
  Mitigation: Establish configuration review processes, document common patterns, and create shared configuration utilities for reusable settings
  Owner: engineering team

## Implementation Notes

- Place all Next.js configuration in next.config.ts files at the root of each Next.js application directory
- Export the configuration object as 'nextConfig' to maintain consistent naming across applications
- Use TypeScript imports for Next.js types (e.g., import type { NextConfig } from 'next') to ensure type safety
- Document any custom configuration extensions or plugins in comments within the configuration file
- Consider creating shared configuration utilities in a common package for settings used across multiple Next.js applications

## Continuation Context


Verify commands:
- find monorepo -name 'next.config.ts' -type f | xargs grep -l 'nextConfig'
- grep -r "libs.core.detected.*next" monorepo/
- find monorepo -path '*/nextjs_app/*/next.config.ts' -type f

Accept when:
- All Next.js applications contain a next.config.ts file with TypeScript typing
- The nextConfig export is present and properly typed in all Next.js configuration files
- Detection pipeline identifies Next.js as a core library (libs.core.detected='next') in public API contexts

## Enforcement

- Verified by: Automated detection pipeline scanning for next.config.ts files and nextConfig exports
- Verified by: Code review verification that new Next.js applications follow configuration standards
- Verified by: CI/CD pipeline checks for TypeScript compilation of Next.js configuration files
- Violation handling: CI build failures for Next.js applications missing required configuration files
- Violation handling: Code review feedback requesting adherence to Next.js configuration standards
- Violation handling: Documentation updates to guide developers toward compliant configuration patterns
- Exception process: Document technical justification for alternative framework or configuration approach
- Exception process: Obtain approval from architecture review board for deviations from Next.js standard
- Exception process: Record exception in ADR system with rationale and scope limitations