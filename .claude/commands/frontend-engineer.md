Generate UI components, audit accessibility, review performance, or analyze state structure.

The target description or file path is: $ARGUMENTS

## Setup

**Default runtime and package manager:** Use **Bun** for all frontend projects unless the project explicitly specifies otherwise. Use `bun install`, `bun add`, and `bun run` — never `npm`, `yarn`, or `pnpm` unless already in use. Lock file is `bun.lockb`. Dockerfiles must install Bun and use it throughout.

Detect the frontend framework by reading `package.json` dependencies. Infer TypeScript usage from `tsconfig.json`. If no package manager is detected, default to Bun. Match the existing file structure and naming conventions before generating any code.

## Modes

### Default — Generate a component
When `$ARGUMENTS` describes a component:
1. Scaffold a complete component with:
   - TypeScript props interface (or JS PropTypes if the project uses JS)
   - Loading state
   - Error state
   - Empty/zero state
   - Semantic HTML structure
   - ARIA attributes where the visual label is absent
   - Co-located styles (CSS module or styled component matching project conventions)
2. Show the component's usage example
3. Flag any behavior that requires a separate custom hook and scaffold it

### `--a11y` — Accessibility audit
For the file or directory in `$ARGUMENTS`:
- Flag: missing `alt` text, non-semantic interactive elements (`<div>` as button/link), missing `<label>` for inputs, missing keyboard handlers on custom controls, missing focus management after modal/dialog transitions
- For each issue: quote the offending line, explain the problem, provide the fix
- Note color contrast issues for manual verification (cannot check programmatically)

### `--perf` — Performance audit
Scan for:
- Inline function or object creation in render without memoization
- `useEffect` with missing or stale dependency arrays
- Prop drilling beyond two levels
- Missing `key` props on lists
- Synchronous operations in the render path
- Large imports where a targeted import would suffice

### `--state` — State audit
For the component tree in `$ARGUMENTS`:
1. Map the current state shape and where it lives
2. Identify: state that can be derived (and should not be stored), duplicated state across components, and state that needs lifting or colocation
3. Propose a revised state structure with rationale

## Output

Generated components must be complete and runnable — no placeholder comments like `// TODO: implement`. Include the import statement needed to use the component.
