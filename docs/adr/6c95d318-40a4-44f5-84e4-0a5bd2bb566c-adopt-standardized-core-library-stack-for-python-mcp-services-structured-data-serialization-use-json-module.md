# Adopt Standardized Core Library Stack for Python MCP Services: Structured Data Serialization Use Json Module

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The MCP services layer requires consistent approaches to logging, configuration management, time handling, unique identifier generation, and data serialization across multiple service modules
- Service modules (event_manager, rsvp) interact with external state, require structured logging for observability, and need environment-based configuration following 12-factor app principles
- Python's standard library provides core utilities (os, typing, datetime, uuid, json) but lacks structured logging capabilities, necessitating a third-party logging solution
- The codebase demonstrates consistent usage patterns across three service modules with 0.90 significance, indicating an established architectural convention rather than ad-hoc choices

## Problem Statement

Service modules in the MCP layer need standardized utilities for common operations (logging, configuration, time handling, ID generation, serialization) to ensure consistent behavior, maintainability, and observability. Without a defined standard library stack, developers may choose incompatible alternatives, leading to fragmented logging formats, inconsistent configuration patterns, and increased cognitive load when working across services.

## Decision

1. SHOULD: Structured data serialization SHOULD use the json module for consistency with external API contracts

## Policy Block

- SHOULD Structured data serialization SHOULD use the json module for consistency with external API contracts

In scope:
- Python service modules in the MCP services layer (monorepo/tlt/mcp_services/*)
- Service components that manage state, interact with external systems, or require observability
- Modules that require environment-based configuration or 12-factor app compliance

Out of scope:
- CLI tools or scripts that may use simpler print-based output
- Utility modules that do not require structured logging
- Test fixtures or mock implementations where alternative libraries may be appropriate
- Non-Python codebases or services in other languages

## Rationale

- The evidence shows consistent adoption across three service modules with 0.90 significance, indicating this is an established architectural pattern rather than coincidental usage
- Loguru provides structured logging with better ergonomics than stdlib logging, supporting observability requirements for service-layer components that manage state and external interactions
- Standardizing on os.getenv for configuration aligns with 12-factor app methodology, enabling environment-specific configuration without code changes
- Using timezone-aware UTC timestamps (datetime.now(timezone.utc)) and UUID v4 for identifiers establishes consistent data handling patterns across distributed service components

## Consequences

Positive:
- Developers experience reduced cognitive load when working across multiple service modules due to consistent library usage and patterns
- Structured logging via loguru enables better observability, debugging, and log aggregation across the MCP services layer
- Type annotations via typing module improve IDE support, enable static analysis, and serve as inline documentation
- Standardized patterns for configuration, timestamps, and ID generation reduce bugs from inconsistent implementations

Negative:
- Introduces a third-party dependency (loguru) that must be maintained and updated, adding to dependency management overhead
- Developers familiar with stdlib logging must learn loguru's API and conventions
- Restricts flexibility for service modules that might benefit from alternative libraries for specific use cases
- Type annotations add verbosity to function signatures and may slow initial development

## Alternatives

- Use Python standard library logging module instead of loguru (rejected)
  Rejected because: Stdlib logging requires more boilerplate for structured logging and lacks the ergonomic API that loguru provides. Evidence shows loguru is already consistently adopted across service modules.
  When valid: For simple scripts or utilities where structured logging is not required and minimizing dependencies is critical
- Allow each service module to choose its own logging and utility libraries (rejected)
  Rejected because: Fragmented library choices increase cognitive load, make cross-service maintenance harder, and prevent consistent observability patterns. The evidence shows standardization is already established.
  When valid: Never valid for MCP service layer; may be acceptable for isolated utility scripts outside the service layer
- Use structlog for structured logging instead of loguru (rejected)
  Rejected because: Evidence shows loguru is already adopted and integrated. Switching would require migration effort without clear architectural benefit given current usage patterns.
  When valid: For new projects or if advanced structured logging features (processors, context binding) become critical requirements

## Risks

- Loguru version updates may introduce breaking changes affecting logging behavior across all service modules
  Mitigation: Pin loguru version in dependency manifest and test logging behavior in CI before upgrading. Review changelog for breaking changes.
  Owner: Backend engineering team
- Developers unfamiliar with loguru may fall back to stdlib logging or print statements, fragmenting observability
  Mitigation: Document loguru usage patterns in service module templates. Include linting rules to detect stdlib logging imports in service layer.
  Owner: Backend engineering team
- Type annotations may become stale as code evolves, providing false confidence in type safety
  Mitigation: Run static type checker in CI to validate type annotations remain accurate. Treat type errors as build failures.
  Owner: Backend engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When creating new MCP service modules, import the standardized library stack at the top of the file following Python import conventions: stdlib imports first (os, typing, datetime, uuid, json), then third-party imports (loguru)
- Configure loguru logger at service initialization to establish consistent log formatting, levels, and output destinations across all service modules
- Use typing.Optional, typing.List, typing.Dict for nullable and collection types in function signatures to maximize type checker effectiveness

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and verify loguru is declared as a dependency for the MCP services package
- Discover the project's static analysis configuration and run the type checker against service modules to verify type annotation coverage
- Discover the project's linting configuration and verify rules detect stdlib logging imports in the MCP services layer

Accept when:
- All MCP service modules import and use the standardized library stack (os, typing, datetime, uuid, json, loguru) without importing stdlib logging
- Static type checker reports no type errors in service module function signatures
- Linting passes with no violations for stdlib logging usage in service layer

## Enforcement

- Verified by: Automated linting in CI pipeline detects non-standard library imports in MCP service modules
- Verified by: Static type checking in CI validates type annotation correctness
- Verified by: Code review checklist includes verification of standardized library usage
- Verified by: Service module templates pre-populate standard library imports
- Violation handling: CI build fails if linting detects stdlib logging imports in service layer
- Violation handling: CI build fails if type checker reports errors in service modules
- Violation handling: Code review blocks merge if non-standard libraries are introduced without architectural justification
- Violation handling: Violations discovered post-merge trigger refactoring tickets to align with standard stack
- Exception process: Developer documents specific technical requirement that standard library stack cannot satisfy
- Exception process: Architectural review evaluates whether requirement justifies exception or if standard stack should be extended
- Exception process: Approved exceptions are documented in service module with rationale comment and tracked in architecture decision log
- Exception process: Exceptions are reviewed quarterly to determine if they should become new standards or be refactored