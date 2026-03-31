# AGENTS.md

## Project Structure

Excalidraw is a **monorepo** with a clear separation between the core library and the application:

- **`packages/excalidraw/`** - Main React component library published to npm as `@excalidraw/excalidraw`
- **`excalidraw-app/`** - Full-featured web application (excalidraw.com) that uses the library
- **`packages/`** - Core packages: `@excalidraw/common`, `@excalidraw/element`, `@excalidraw/math`, `@excalidraw/utils`
- **`examples/`** - Integration examples (NextJS, browser script)

## Development Workflow

1. **Package Development**: Work in `packages/*` for editor features
2. **App Development**: Work in `excalidraw-app/` for app-specific features
3. **Testing**: Always run `corepack yarn test:update` before committing
4. **Type Safety**: Use `corepack yarn test:typecheck` to verify TypeScript

## Development Commands

```bash
corepack yarn test:typecheck  # TypeScript type checking
corepack yarn test:update     # Run tests (with snapshot updates)
corepack yarn test:code       # Lint validation
corepack yarn fix             # Auto-fix formatting and linting issues
```

## Architecture Notes

### Package System

- Uses Yarn workspaces for monorepo management
- Internal packages use path aliases (see `vitest.config.mts`)
- Build system uses esbuild for packages, Vite for the app
- TypeScript throughout with strict configuration

## Rules to Apply

Apply the following rule files whenever you touch matching scope:

- **Architecture** (`.cursor/rules/architecture.mdc`)
  - Keep editor state changes in the existing action flow (`actionManager.dispatch()`).
  - Keep drawing in the Canvas 2D render pipeline (`Scene -> renderScene() -> canvas context`).
  - Avoid introducing alternate state/render libraries or unapproved dependencies.
- **Conventions** (`.cursor/rules/conventions.mdc`)
  - Use functional components and hooks only.
  - Prefer named exports and strict TypeScript (`no any`, `no @ts-ignore`).
  - Keep naming conventions: kebab-case utilities, PascalCase component files, `{ComponentName}Props`.
- **Security** (`.cursor/rules/security-checks.mdc`)
  - Never hardcode credentials or log secrets.
  - Treat imported/network/URL/user input as untrusted and validate/sanitize before use.
  - Block unsafe URL schemes (`javascript:`, `data:`) and avoid raw HTML injection paths.
- **Testing** (`.cursor/rules/testing.mdc`)
  - Run required test and quality checks based on change scope.
  - Update snapshots only for intentional UI/behavior changes.
  - Do not weaken assertions or skip tests without explicit approval.

### When To Apply What

- **Changes in `packages/*` editor/component logic**
  - Apply architecture + conventions + testing rules.
  - Run: `corepack yarn test:update`, `corepack yarn test:typecheck`, and `corepack yarn test:code`.
- **Changes in `excalidraw-app/*` behavior**
  - Apply conventions + security + testing rules (plus architecture when render/state paths are touched).
  - Run: `corepack yarn test:app --watch=false`, `corepack yarn test:typecheck`, and `corepack yarn test:code`.
- **Changes to dependencies (`package.json`/lockfile)**
  - Apply architecture dependency constraints + security supply-chain checks.
  - Run: `corepack yarn npm audit --recursive` and appropriate test suite for impacted scope.
- **Broad or release-sensitive changes**
  - Run full suite: `corepack yarn test:all`.

## Verification Checklist

Before merge or handoff, verify:

- `corepack yarn test:typecheck` passes.
- `corepack yarn test:code` passes for touched TS/TSX scope.
- `corepack yarn test:update` is run when snapshots/behavior changed.
- `corepack yarn test:app --watch=false` is run for app-level behavior changes.
- `corepack yarn test:all` is run for broad or risky changes.
- `corepack yarn npm audit --recursive` is run when dependency manifests or lockfiles changed.
- No `@ts-ignore`, unnecessary `any`, weakened assertions, or skipped tests were introduced to force passing checks.
