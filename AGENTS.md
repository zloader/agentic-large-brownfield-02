# AGENTS.md — Excalidraw Coding Agent Instructions

> Instructions for AI coding agents (Copilot, Codex, Cursor, Devin, etc.) working in this
> repository. Read this file first at the start of every session.
>
> For context on the session's current sprint, open `docs/memory/activeContext.md`.
> For the in-app AI agent catalog, see `docs/memory/agents-catalog.md`.

---

## Project Overview

Excalidraw is a **client-side, offline-first** whiteboard application. This monorepo contains:

- `packages/common` — shared utilities
- `packages/math` — geometry / math primitives
- `packages/element` — element types, mutations, store
- `packages/excalidraw` — core library (published to npm)
- `excalidraw-app` — the production web app (`excalidraw.com`)
- `examples/` — integration examples

There is **no application server** in this repository. All runtime logic runs in the browser.

---

## Dev Environment

**Requirements:** Node.js ≥ 18, yarn v1 (classic)

```bash
yarn install          # install all workspace dependencies
```

> **Use `yarn` exclusively.** Do not use `npm` or `pnpm` — they will produce incorrect
> lockfile state in this workspace.

---

## Monorepo Build Order

**Critical:** packages have a hard dependency chain. Always build in this order:

```
common → math → element → excalidraw → excalidraw-app
```

```bash
yarn build:packages   # builds common, math, element, excalidraw in order
yarn start            # starts the excalidraw-app dev server (run after build:packages)
```

Individual package builds (when targeting a specific package):

```bash
yarn build:common
yarn build:math
yarn build:element
yarn build:excalidraw
yarn build:app        # production build of excalidraw-app
```

---

## Build & Test Commands

```bash
yarn test             # run vitest (excalidraw-app unit tests)
yarn test:typecheck   # tsc type checking across all packages
yarn test:code        # eslint (0 warnings allowed)
yarn test:other       # prettier format check
yarn test:all         # runs typecheck + code + other + app tests
yarn fix              # auto-fix lint and formatting issues
```

> After **any non-documentation change**, run `yarn test` and `yarn test:typecheck` before
> committing to confirm no regressions.

> `noEmit: true` in `tsconfig.json` — TypeScript is used for type-checking only; Vite
> handles compilation.

---

## Protected Files

**Never modify these files without explicit user approval and a full test suite pass:**

| File                                       | Reason                             |
|:-------------------------------------------|:-----------------------------------|
| `packages/excalidraw/scene/Renderer.ts`    | Core render pipeline               |
| `packages/excalidraw/data/restore.ts`      | File format backward compatibility |
| `packages/excalidraw/actions/manager.tsx`  | Action system                      |
| `packages/excalidraw/types.ts`             | Core type definitions              |

Changes to protected files require: full understanding of dependents + `yarn test:all` pass +
manual QA verification.

---

## Code Conventions

### TypeScript
- Strict mode — no `any`, no `@ts-ignore`
- Prefer `type` over `interface` for simple shapes
- Import types: `import type { X } from "..."`
- Use optional chaining (`?.`) and nullish coalescing (`??`)
- Prefer `const` / `readonly`; avoid mutation

### React
- Functional components + hooks only — no class components
- Props type named `{ComponentName}Props`
- Named exports only — no default exports
- CSS modules for component styling
- No conditional hooks

### Files
- kebab-case filenames: `element-utils.ts`
- PascalCase component files: `LayerUI.tsx`
- Colocated tests: `ComponentName.test.tsx`

### Naming
- PascalCase: component names, interfaces, type aliases
- camelCase: variables, functions, methods
- ALL_CAPS: constants

### Element mutations
- Always use `mutateElement()` (increments `version`/`versionNonce`) or `newElementWith()`
- Never assign element properties directly
- Raw `mutateElement()` bypasses history and collab sync — use only when intentional

### Math / geometry
- Always use the `Point` type from `packages/math/src/types.ts` instead of `{ x, y }`

---

## Architecture Constraints

- **No Redux / MobX.** State: Jotai atoms (cross-component) + `React.useState` (local).
- **No direct DOM mutation.** Canvas changes go through `requestAnimationFrame` + Canvas 2D API.
- **No new npm packages** without explicit approval — check `packages/utils/` first.
- **Encryption boundary.** All data leaving the client must be AES-GCM encrypted. Keys live
  exclusively in URL fragments — never in server storage, logs, or transmitted payloads.
- **Client-side only.** Node.js ≥ 18 is for the build toolchain only; no server runtime.

---

## Memory Bank

This repository uses a **memory bank** pattern (`docs/memory/`) for AI development context.
Read these files at the start of each session to avoid re-discovering context:

| File | Purpose |
|------|---------|
| `docs/memory/activeContext.md` | Current sprint, ongoing work, known issues |
| `docs/memory/systemPatterns.md` | Architectural patterns — read before touching core subsystems |
| `docs/memory/decisionLog.md` | Implicit constraints and high-risk clusters |
| `docs/memory/techContext.md` | Tech stack, dependencies, env variables |
| `docs/memory/productContext.md` | Product goals, user flows, pain points |
| `docs/memory/projectbrief.md` | Project scope and boundaries |
| `docs/memory/progress.md` | Completed work log |

> Update `docs/memory/activeContext.md` at sprint boundaries.
> Append to `docs/memory/progress.md` after completing significant work.
> Files in `docs/scopes/` (Repomix snapshots) go stale quickly — prefer live source files.

---

## Before Committing

1. Run `yarn fix` to auto-fix lint and formatting
2. Run `yarn test && yarn test:typecheck` — both must pass
3. Include the following trailer in every commit message:

---

*Last updated: 2026-03-29. Update this file when dev environment, build commands,
conventions, or protected files change.*
