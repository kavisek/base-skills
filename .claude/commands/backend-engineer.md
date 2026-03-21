Generate API endpoints, review handler logic, audit auth, or design service interfaces.

The target description or resource is: $ARGUMENTS

## Setup

**Default framework:** Use **FastAPI** for all backend services unless the project or `$ARGUMENTS` explicitly specifies a different framework.

**Testing:** Use **pytest** for all backend tests. **Linting:** Use **black** for formatting. Both must be included in the project's dev dependencies and run inside the container. Do not substitute Flask, Django, Express, or another framework without a stated reason.

Detect the backend framework by reading `package.json`, `pyproject.toml`, `requirements.txt`, or equivalent config files. If no framework is detected, default to FastAPI. Match the existing project structure for file placement and naming before generating code.

## Modes

### Default — Generate an endpoint
When `$ARGUMENTS` describes a resource or operation:
1. Scaffold a complete endpoint including:
   - Request schema / type definition
   - Response schema / type definition
   - Handler function (thin — orchestration only)
   - Service function (business logic, calls data layer)
   - Input validation at the boundary
   - Error handling with correct HTTP status codes
   - Auth middleware reference (note which middleware to apply; do not omit it)
   - A test stub for the happy path and one error case
2. End with a **Security Checklist**:
   - [ ] Input validated and sanitized
   - [ ] Response does not leak internal details
   - [ ] Auth middleware applied
   - [ ] Rate limit strategy documented

### `--review` — Review existing handlers
For the file or directory in `$ARGUMENTS`:
- Flag: incorrect HTTP status codes, missing error handling, business logic in handlers, missing input validation, routes missing auth middleware, raw exceptions exposed to the client
- For each issue: quote the line, explain the problem, provide the corrected version

### `--auth` — Auth audit
Scan the route/handler files for:
- Routes missing authentication middleware
- Authentication and authorization logic conflated in the same function
- Hardcoded role or permission strings that should be policy-driven
- Tokens or credentials appearing in log statements

### `--service` — Design a service interface
Given a description of a domain operation in `$ARGUMENTS`:
1. Design the service interface before any implementation:
   - Input type with validation rules noted
   - Output type
   - Named error types (not generic exceptions)
   - Docstring describing the business rule this service enforces
2. Do not implement the service body — present the interface for review first

## Output

Generated code must be complete and match the project's framework conventions. No placeholder comments. Include all necessary imports.
