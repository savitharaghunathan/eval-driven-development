# API Service Archetype

## Signature Patterns

API endpoints, tool definitions, MCP server, function-calling interface, request-response service consumed by other agents or systems.

**Examples:** MCP tool server, REST API, function-calling endpoint, webhook handler, data access layer.

## Extraction Checklist

When analyzing a spec for API service systems, look for:

- API contract (OpenAPI spec, MCP tool definitions, schema)
- Parameter validation rules
- Error response format and codes
- Authentication/authorization model
- Rate limiting and throttling behavior
- Idempotency guarantees
- Backward compatibility constraints
- Timeout and retry expectations

## Default Eval Dimensions

- Contract compliance (responses match declared schema)
- Error format consistency (all errors follow the same envelope)
- Parameter validation coverage (rejects bad input, accepts good input)
- Idempotency (repeated calls produce same result)
- Latency percentiles (p50, p95, p99)
- Backward compatibility (old clients still work after changes)
- Auth enforcement (unauthorized requests rejected)

## Default Grader Patterns

| Requirement Type | Grader Type | Notes |
| ---------------- | ----------- | ----- |
| Contract compliance | code-based | JSON schema validation on responses |
| Error format consistency | code-based | Check error envelope structure |
| Parameter validation | code-based | Send invalid params, check rejection |
| Idempotency | outcome verification | Send same request twice, compare |
| Latency percentiles | code-based | Measure timing |
| Backward compatibility | code-based | Replay old client requests |
| Auth enforcement | code-based | Send unauthenticated requests, check 401/403 |

## Common Failure Modes

- API returns 200 with an error message in the body instead of using proper HTTP status codes
- Error responses use inconsistent formats (sometimes JSON, sometimes plain text)
- Parameter validation rejects valid edge cases (empty strings, Unicode, large numbers)
- Idempotency keys are ignored or handled inconsistently
- Rate limiting returns unhelpful error messages (no Retry-After header)
- Breaking changes deployed without version bump (silent backward incompatibility)
- Auth middleware has gaps (some endpoints unprotected)

## Example Eval Tasks

```yaml
- id: contract-compliance
  requirement: API responses match the declared schema
  description: Call each endpoint with valid parameters and validate response against OpenAPI/MCP schema
  input: Valid request to each endpoint
  expected_behavior: Every response passes JSON schema validation for that endpoint
  reference_solution: GET /users/123 returns {"id": 123, "name": "Alice", "email": "alice@example.com"} matching the User schema
  grader_type: code
  grader_logic: For each endpoint, validate response body against its declared JSON schema. PASS if all fields present and types match. FAIL on any schema violation.
  category: capability
  priority: P0

- id: error-format-consistency
  requirement: All error responses follow the same envelope format
  description: Trigger errors across multiple endpoints (400, 401, 404, 500) and check format consistency
  input: Invalid requests designed to trigger each error code
  expected_behavior: All errors return {"error": {"code": <int>, "message": <string>}} format
  reference_solution: POST /users with missing required field returns {"error": {"code": 400, "message": "Field 'name' is required"}}
  grader_type: code
  grader_logic: Trigger 400, 401, 404, 422 errors. Check each response has top-level "error" object with "code" (int) and "message" (string). PASS if all consistent.
  category: capability
  priority: P1
```
