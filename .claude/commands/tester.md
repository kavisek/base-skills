Generate tests, audit existing tests, or run a TDD cycle for the target file or feature.

**Container requirement:** All test execution happens inside a container. Always verify or scaffold a `Dockerfile` (or `Dockerfile.test`) and a `docker-compose.yml` with a `test` service before generating test commands. Never suggest running tests directly on the host machine.

## Setup Check (always run first)

Before anything else:
1. Check for a `Dockerfile` or `Dockerfile.test` in the project root
2. Check for a `docker-compose.yml` with a `test` service
3. If either is missing, generate them before proceeding — ask the user to confirm the runtime and framework
4. Identify the test framework in use (pytest, Jest, Vitest, Mocha, etc.) by reading `package.json`, `pyproject.toml`, or config files

## Modes

### Default — Generate tests
When `$ARGUMENTS` provides a file path or feature description, generate a complete test file:
- Detect whether a test counterpart already exists; if so, extend it rather than replace it
- One test per distinct behavior (not per function)
- Cover: happy path, error path, edge cases (null/empty/boundary values)
- Follow Arrange-Act-Assert structure with blank lines between phases
- Test names read as sentences describing behavior
- Include a `docker compose run --rm test` command at the end showing how to run the new tests

### `--audit` — Audit existing tests
For the file or directory in `$ARGUMENTS`:
- Find all test files and report:
  - Code paths with no test coverage
  - Tests that violate AAA (logic, conditionals, or loops inside test body)
  - Tests with overly broad assertions (`assert True`, `expect(result).toBeDefined()`)
  - Shared mutable state between tests
- Produce a prioritized fix list

### `--tdd` — TDD cycle
Given a behavior description in `$ARGUMENTS`:
1. Write a single failing test that specifies the behavior
2. Show the `docker compose run --rm test` command to confirm it fails (red)
3. Pause and ask the user to implement — do not implement for them
4. When the user returns, review their implementation and suggest the refactor step

### `--container` — Scaffold container setup
Generate or fix the container setup for testing:
- `Dockerfile.test` with the correct runtime version and dependency install step
- `docker-compose.yml` with a `test` service and any required dependent services (db, cache, etc.)
- A `Makefile` target: `make test` → `docker compose run --rm test`

## Output

Always end with the exact `docker compose` command needed to run the tests.
