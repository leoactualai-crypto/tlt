# Adopt .get() Dictionary Access Pattern for Integration Testing and External API Response Handling: Get Method Specify

Status: proposed
Date: 2024-01-09
Deciders: Detection Pipeline (automated)

## Activation

This ADR is always active for all internal API implementations, integration test suites, and external client boundary code within the monorepo.

## Context

- The monorepo contains multiple MCP services (rsvp, guild_manager, event_manager, photo_vibe_check) and Discord adapter commands that interact through internal APIs and external HTTP clients
- Integration testing requires safe extraction of response data from dictionaries where keys may be absent due to API version mismatches, partial failures, or optional fields
- External client boundaries (httpx, requests) return responses as dictionaries where field presence varies based on HTTP status codes, service availability, and API contract evolution
- The codebase demonstrates consistent use of .get() method with default values across 26 files handling response.get('message'), result.get('success'), event.get('guild_id'), and similar patterns
- Services use loguru for logging and fastmcp for MCP protocol implementation, requiring defensive dictionary access to prevent KeyError exceptions in production

## Problem Statement

Internal APIs and external service integrations return dictionary responses with variable key presence, creating risk of KeyError exceptions when accessing fields that may be absent due to API evolution, partial failures, optional fields, or service degradation. Direct dictionary key access (dict['key']) causes runtime failures, while .get() method provides safe extraction with default values, enabling graceful degradation and robust error handling in integration tests and production code.

## Decision

1. MUST: The .get() method MUST specify an appropriate default value (e.g., None, empty string, 0, empty dict, False) that enables safe downstream processing

## Policy Block

- MUST The .get() method MUST specify an appropriate default value (e.g., None, empty string, 0, empty dict, False) that enables safe downstream processing

In scope:
- All MCP service tools (rsvp/tools.py, guild_manager/tools.py, photo_vibe_check/tools.py, event_manager/main.py)
- Discord adapter command handlers (info/handler.py, update/handler.py, list/handler.py, vibe/handler.py, delete/handler.py)
- External client implementations (discord_adapter/clients/tlt_client.py)
- Integration test code accessing API responses or service results
- Photo processor and workflow state handling (photo_processor.py)
- Agent state management accessing lifecycle dictionaries (ambient_event_agent/state/state.py)

Out of scope:
- Pydantic BaseModel field access where validation guarantees field presence
- Internal function parameters with type hints guaranteeing dictionary structure
- Configuration dictionaries loaded from validated schemas
- Dictionary literals created within the same function scope

Exceptions:
- EXC-001: Direct key access is permitted when the dictionary is a Pydantic model's .dict() or .model_dump() output and the field is required (not Optional)
- EXC-002: Direct key access is permitted for dictionary keys that are explicitly validated with 'if key in dict' guard immediately before access

## Rationale

- Evidence shows 26 files consistently using .get() pattern for response.get('message'), result.get('success'), event.get('guild_id'), stats.get('total_rsvps', 0), indicating established practice for safe dictionary access
- External client boundaries using httpx and requests return responses where field presence varies by HTTP status code (2xx vs 4xx vs 5xx), requiring defensive access to prevent KeyError in error paths
- Integration testing facet detection across multiple handlers demonstrates need for test resilience when API contracts evolve or services return partial responses
- The pattern enables graceful degradation: services can continue operating with default values when optional fields are absent, improving system resilience

## Consequences

Positive:
- Eliminates KeyError exceptions in production when API responses omit optional or conditional fields
- Integration tests remain stable across API version changes and partial service failures
- Code explicitly documents expected response structure through default values
- Enables graceful degradation when external services return incomplete data
- Reduces coupling between services by not requiring strict response contracts

Negative:
- Silent failures possible if default values mask genuine API contract violations that should be caught
- Increased verbosity compared to direct key access, especially for deeply nested structures
- May hide bugs where missing keys indicate upstream service failures that should be surfaced
- Default values must be carefully chosen to avoid incorrect behavior when keys are legitimately absent

## Alternatives

- Use direct dictionary key access (dict['key']) and catch KeyError exceptions at call sites (rejected)
  Rejected because: Exception handling adds boilerplate at every call site, makes code harder to read, and exception stack traces are more expensive than .get() default returns. Evidence shows no try/except KeyError patterns in the 26 files.
  When valid: Only valid for critical fields where absence should halt execution and trigger alerts
- Enforce strict API contracts with Pydantic validation at service boundaries, failing fast on missing required fields (deferred)
  Rejected because: Not rejected but complementary. Pydantic validation should be used for required fields, while .get() handles optional fields and backward compatibility. Evidence shows both patterns coexist (BaseModel classes and .get() usage).
  When valid: Should be used in combination with .get() pattern: Pydantic for required fields, .get() for optional fields
- Use TypedDict with total=False for optional fields and rely on type checkers to enforce safe access (rejected)
  Rejected because: TypedDict provides static type checking but does not prevent runtime KeyError. Evidence shows runtime dictionary access from external APIs (httpx, requests) where static types cannot guarantee key presence.
  When valid: Useful as complementary documentation but does not replace runtime .get() safety

## Risks

- Default values may mask API contract violations where missing keys indicate upstream service failures that should be detected and alerted
  Mitigation: Log warnings when critical fields are missing using pattern: if 'key' not in response: logger.warning(). Combine with monitoring to detect degraded API responses.
  Owner: Service reliability team
- Inconsistent default values across codebase may lead to different behavior when same API field is accessed in multiple locations
  Mitigation: Document expected default values in API client classes. Create shared constants for common defaults (e.g., DEFAULT_GUILD_ID = 'unknown').
  Owner: API integration team
- Chained .get() calls (dict.get('a', {}).get('b')) may hide None propagation bugs if intermediate default is not empty dict
  Mitigation: Code review checklist item: verify chained .get() uses {} or [] as intermediate defaults. Add linting rule to detect .get().get() without dict/list defaults.
  Owner: Engineering team

## Implementation Notes

- For external API responses (httpx, requests), always use .get() with defaults that enable error path execution: result.get('success', False), result.get('error', 'Unknown error')
- For nested dictionary access, chain .get() with appropriate container defaults: state.get('lifecycles', {}).get(task_id) prevents AttributeError
- In integration tests, use .get() with explicit defaults to document expected response structure: assert response.get('status') == 'success'
- When logging response data, use .get() to prevent log formatting failures: logger.info(f"Guild: {result.get('guild_id', 'unknown')}")
- For CloudEvent data extraction, use .get() since event data structure varies by event type: event.get('guild_id'), event.get('message_id')

## Continuation Context


Verify commands:
- grep -r "\['[a-zA-Z_]*'\]" monorepo/tlt/mcp_services --include='*.py' | grep -v test | grep -v '.get(' | wc -l
- grep -r "\.get(" monorepo/tlt/mcp_services monorepo/tlt/adapters/discord_adapter --include='*.py' | grep -E "(response|result|event|data|state)\.get\(" | wc -l
- python -m pytest monorepo/tlt/tests/integration -v --tb=short 2>&1 | grep -i keyerror | wc -l

Accept when:
- Direct dictionary key access count in service and adapter code is less than 5% of total dictionary accesses (excluding Pydantic model access)
- .get() method usage count for response/result/event/state dictionaries exceeds 95% of accesses in integration boundary code
- Integration test suite runs without KeyError exceptions when external services return partial or error responses

## Enforcement

- Verified by: Code review checklist requiring .get() usage for all external API response handling
- Verified by: Integration test suite validation that no KeyError exceptions occur during normal and error path execution
- Verified by: Static analysis grep patterns in CI checking for direct key access in service and adapter modules
- Violation handling: CI pipeline fails if grep detects direct dictionary key access patterns in new code touching external API boundaries
- Violation handling: Code review blocks merge if direct key access is used without explicit guard clause (if key in dict)
- Violation handling: Production KeyError exceptions trigger incident review to add .get() usage and integration test coverage
- Exception process: Developer documents in code comment why direct key access is safe (e.g., Pydantic model field, validated guard clause)
- Exception process: Code reviewer verifies exception justification and approves with explicit comment in PR
- Exception process: Exception is logged in ADR exceptions registry with file path, line number, and justification