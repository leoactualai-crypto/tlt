# Environment Variable Retrieval for API Credentials: Environment Variable Names Credentials Follow Consistent

Status: proposed
Date: 2025-01-17
Deciders: Detection Pipeline (automated)

## Context

- The RSVP service integrates with the OpenAI API to interpret emoji reactions for event RSVPs, requiring authentication via API key
- API credentials must not be hardcoded in source code to prevent exposure in version control systems and enable environment-specific configuration
- The codebase uses runtime environment variable lookup as the mechanism for retrieving external service credentials
- This pattern was observed in the context of OpenAI API integration where the service retrieves 'OPENAI_API_KEY' from the environment at runtime

## Problem Statement

Services that integrate with external APIs requiring authentication credentials need a secure method to retrieve secrets without embedding them in source code. Hardcoded credentials create security vulnerabilities through version control exposure, make credential rotation difficult, and prevent environment-specific configuration. A standardized approach to secret retrieval is needed that maintains security while enabling operational flexibility.

## Decision

1. SHOULD: Environment variable names for API credentials SHOULD follow a consistent naming convention that identifies the service and credential type

## Policy Block

- SHOULD Environment variable names for API credentials SHOULD follow a consistent naming convention that identifies the service and credential type

In scope:
- Backend services that authenticate with external APIs
- Services that require secret credentials for third-party integrations
- Components that need environment-specific configuration for authentication

Out of scope:
- Frontend code where secrets cannot be safely stored
- Public client applications where credentials would be exposed
- Internal service-to-service communication using mutual TLS or service mesh authentication
- Development tooling that uses dedicated secret management services

## Rationale

- Environment variable retrieval prevents credential exposure in version control systems by keeping secrets out of source code
- Runtime environment variable lookup enables environment-specific configuration, allowing different credentials for development, staging, and production without code changes
- This pattern was observed in the RSVP service's OpenAI API integration, establishing a precedent for how services should handle external API authentication
- The approach provides a baseline security posture while remaining simple to implement and operationally manageable

## Consequences

Positive:
- Credentials are never committed to version control, reducing the risk of accidental exposure
- Environment-specific configuration is straightforward, enabling different credentials per deployment environment
- Credential rotation can occur without code changes or redeployment
- The pattern is simple to implement using standard library functions without additional dependencies

Negative:
- Environment variables can be exposed through process listings, logs, or error messages if not handled carefully
- No built-in secret rotation, versioning, or audit logging capabilities
- Secrets must be manually configured in each deployment environment
- Limited access control compared to dedicated secret management services

## Alternatives

- Hardcode credentials directly in source files (rejected)
  Rejected because: Creates severe security vulnerability by exposing credentials in version control and preventing environment-specific configuration
  When valid: Never valid for production systems
- Use dedicated secret management service with SDK integration (deferred)
  Rejected because: Not rejected, but not currently adopted. Would provide superior security features (rotation, audit, access control) at the cost of additional infrastructure complexity
  When valid: When security requirements demand centralized secret management, audit logging, or automated rotation
- Store credentials in configuration files outside version control (rejected)
  Rejected because: Configuration files are harder to manage across environments and still require careful handling to prevent accidental commits
  When valid: Legacy systems where environment variable support is limited

## Risks

- Environment variables may be logged or exposed in error messages, stack traces, or process listings
  Mitigation: Implement logging filters to redact credential values, handle credential retrieval errors without exposing values, and restrict process inspection capabilities in production
  Owner: Backend engineering team
- Missing environment variables cause runtime failures that may not be detected until deployment
  Mitigation: Implement startup validation that checks for required environment variables before service initialization, document required variables clearly, and include environment variable checks in deployment verification
  Owner: DevOps and backend engineering teams
- No centralized audit trail for secret access or rotation events
  Mitigation: Document credential rotation procedures, implement application-level logging for authentication events (without logging credential values), and consider migration to dedicated secret management for high-security requirements
  Owner: Security and backend engineering teams

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Validate that all required environment variables are present during service initialization, before attempting to use external APIs, to fail fast with clear error messages
- Document required environment variables in service README files, deployment guides, and infrastructure-as-code templates to ensure proper configuration across environments
- Consider implementing a configuration validation module that centralizes environment variable retrieval and validation logic to ensure consistent error handling

## Continuation Context


Verify commands:
- Discover and execute the project's static analysis or linting configuration to detect hardcoded credential patterns in source files
- Discover and execute the project's test suite to verify that services handle missing environment variables gracefully with appropriate error messages
- Inspect the repository to confirm no files contain credential values and that credential-related files are properly listed in version control ignore configurations

Accept when:
- Static analysis confirms no hardcoded credentials exist in source files
- Service initialization tests verify that missing required environment variables produce clear error messages before attempting API calls
- Version control ignore configurations prevent credential files from being committed

## Enforcement

- Verified by: Pre-commit hooks that scan for credential patterns in staged files
- Verified by: Code review checklist items verifying environment variable usage for new external API integrations
- Verified by: Static analysis in CI pipeline that detects potential hardcoded secrets
- Violation handling: Pre-commit hooks block commits containing potential credentials
- Violation handling: CI pipeline fails builds that contain hardcoded credential patterns
- Violation handling: Code review requires revision before merge if credentials are not properly externalized
- Exception process: No exceptions permitted for production code
- Exception process: Test fixtures may use mock credentials clearly marked as non-functional examples
- Exception process: Documentation examples must use placeholder values with clear indication they are not real credentials