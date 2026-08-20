# Adopt Loguru for Logging in MCP Gateway Service: Modules Requiring Logging Import Logger Instance

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The MCP gateway service requires structured logging for service initialization, request handling, and error reporting across multiple modules.
- Python's standard library logging module requires significant boilerplate for configuration, handler setup, and formatter definition.
- The gateway service operates in both stdio transport mode and HTTP server mode, requiring consistent logging behavior across execution contexts.
- Log level configuration is externalized via environment variables, requiring runtime configuration support.

## Problem Statement

The MCP gateway service needs a logging solution that minimizes configuration overhead, provides consistent structured output across service boundaries, and supports runtime configuration through environment variables without requiring extensive setup code in each module.

## Decision

1. MUST: Modules requiring logging MUST import the logger instance using the canonical import pattern from the Loguru package.

## Policy Block

- MUST Modules requiring logging MUST import the logger instance using the canonical import pattern from the Loguru package.

In scope:
- All modules within the tlt.mcp_services.gateway package
- Service entry points that initialize the gateway application
- Resource handlers and request processing modules
- Components that require operational visibility or debugging output

Out of scope:
- Third-party libraries that implement their own logging (these may use stdlib logging internally)
- Test fixtures that mock logging behavior
- Modules outside the MCP gateway service boundary that may have different logging requirements

## Rationale

- The evidence shows consistent adoption of Loguru across gateway service modules (main.py and resources.py) with significance 0.89, indicating an established architectural choice.
- Loguru eliminates the configuration boilerplate required by Python's standard library logging module, reducing code complexity in service initialization paths.
- The import pattern 'from loguru import logger' provides immediate access to a pre-configured logger instance without requiring handler or formatter setup.
- The pattern supports runtime configuration through environment variables while maintaining simple call-site syntax for logging operations.

## Consequences

Positive:
- Reduced boilerplate code in modules requiring logging functionality - no handler or formatter configuration needed.
- Consistent logging output format across all gateway service modules without per-module configuration.
- Simplified developer experience with intuitive API that requires minimal learning curve.
- Automatic context capture and formatting reduces manual string construction in log messages.

Negative:
- Introduces external dependency on third-party logging library, adding to dependency surface area.
- Developers familiar with stdlib logging patterns must learn Loguru-specific API and configuration approaches.
- Potential integration friction with libraries or frameworks that expect stdlib logging handlers.
- Global logger state may complicate testing scenarios that require isolated logging contexts.

## Alternatives

- Use Python standard library logging module with centralized configuration (rejected)
  Rejected because: Requires significant boilerplate for handler setup, formatter configuration, and logger hierarchy management in each service module, increasing maintenance burden.
  When valid: When integration with existing stdlib logging infrastructure is mandatory, or when zero external dependencies is a hard requirement.
- Use structlog for structured logging with processor chains (rejected)
  Rejected because: Adds complexity through processor chain configuration and context binding patterns that exceed the gateway service's current logging requirements.
  When valid: When advanced structured logging features like context processors, log enrichment pipelines, or machine-readable output formats are required.
- Implement custom logging wrapper around stdlib logging (rejected)
  Rejected because: Creates maintenance burden for custom logging infrastructure and duplicates functionality available in mature third-party libraries.
  When valid: When highly specific logging behavior is required that cannot be achieved through existing library configuration.

## Risks

- Loguru's global logger state may cause test isolation issues if multiple tests configure logging differently.
  Mitigation: Implement test fixtures that capture and restore logger configuration state, or use Loguru's remove() and add() methods to manage handlers in test setup/teardown.
  Owner: Gateway service engineering team
- Third-party libraries using stdlib logging may produce inconsistent output format compared to Loguru-logged messages.
  Mitigation: Configure Loguru to intercept stdlib logging calls using the intercept handler pattern documented in the library's integration guide.
  Owner: Gateway service engineering team
- Version updates to Loguru may introduce breaking API changes affecting logging call sites across the service.
  Mitigation: Pin Loguru version in dependency manifest with explicit version constraints, and review changelog before upgrading.
  Owner: Gateway service engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Import the logger instance at module level to ensure consistent access across all functions and classes within the module.
- When configuring log levels from environment variables, implement validation and fallback to default levels to prevent runtime errors from invalid configuration.
- For error logging scenarios, use the logger's exception() method to automatically capture and format stack traces.

## Continuation Context


Verify commands:
- Discover the project's dependency manifest and locate the logging library declaration to confirm it is declared as a project dependency.
- Discover and execute the project's import validation or linting tooling to verify that logging imports follow the canonical pattern across gateway service modules.
- Discover and execute the project's test suite for the gateway service to verify that logging calls do not cause test failures or output pollution.

Accept when:
- All gateway service modules import the logger using the canonical import pattern from the logging library.
- The dependency manifest declares the logging library with appropriate version constraints.
- Service initialization and request handling code produces structured log output without configuration errors.

## Enforcement

- Verified by: Code review verification that new gateway service modules use the adopted logging library.
- Verified by: Static analysis or import linting to detect usage of alternative logging implementations.
- Verified by: CI pipeline checks that verify logging library presence in dependency lock artifact.
- Violation handling: Pull requests introducing stdlib logging or alternative logging libraries in gateway service code are rejected during code review.
- Violation handling: Automated linting failures block CI pipeline progression when non-compliant logging imports are detected.
- Violation handling: Existing violations are tracked as technical debt items and prioritized for remediation.
- Exception process: Exceptions require written justification documenting why the adopted logging library cannot satisfy the use case.
- Exception process: Exception requests are reviewed by the gateway service technical lead or architecture review board.
- Exception process: Approved exceptions are documented with scope boundaries and expiration conditions.