# AGENTS.md

Guide for AI assistants and contributors working in this **Excalidraw monorepo**. Prefer **published MDX** under [`dev-docs/docs/`](dev-docs/docs/) for embedding/API and integration details before inferring from code alone.

## Project overview

[Excalidraw](https://excalidraw.com) is a virtual whiteboard: diagramming, collaboration hooks in the app, and an embeddable **React** library (`@excalidraw/excalidraw`). This repository is a **Yarn workspaces monorepo**: shared packages under `packages/`, the full web app in `excalidraw-app/`, developer docs in `dev-docs/`, and small **examples/** for integration patterns. Library code and app code have different release and security boundaries—treat the npm package surface as distinct from excalidraw.com features.

## Tech stack

| Area | Choice |
|------|--------|
| Language | **TypeScript** (strict) |
| UI | **React** |
| App dev / build | **Vite** (`excalidraw-app`) |
| Package builds | **esbuild** (+ Sass where needed); `@excalidraw/utils` via `scripts/buildUtils.js` |
| Package manager | **Yarn 1** `1.22.22` |
| Runtime | **Node** `>=18.0.0` |
| Tests | **Vitest** (`vitest.config.mts`, path aliases for workspaces) |
| Lint / format | **ESLint**, **Prettier** (`yarn fix`) |

Authoritative versions, Docker, and the full script table: [`docs/memory/techContext.md`](docs/memory/techContext.md).

## Project structure

- **`dev-docs/`** — Developer documentation site source; MDX under `dev-docs/docs/` (API, contributing, mermaid package, codebase topics).
- **`packages/excalidraw/`** — Main library published as `@excalidraw/excalidraw` (see [`docs/memory/systemPatterns.md`](docs/memory/systemPatterns.md) for boundaries).
- **`excalidraw-app/`** — Full product app (Vite) consuming workspace packages.
- **`packages/*`** — `@excalidraw/common`, `@excalidraw/math`, `@excalidraw/element`, `@excalidraw/utils` (`0.1.2`, separate build path; no root `build:utils`), `@excalidraw/excalidraw`.
- **`examples/`** — Next.js and browser integration examples.

**Workspaces** (root `package.json`): `excalidraw-app`, `packages/*`, `examples/*`.

**Package build order:** `yarn build:packages` → **common → math → element → excalidraw**. For utils only: `yarn --cwd ./packages/utils build:esm`.

## Key commands

```bash
yarn start              # Dev server (Vite in excalidraw-app)
yarn build              # App production build
yarn build:packages     # common → math → element → excalidraw

yarn test               # Vitest (same as test:app)
yarn test:app           # Vitest
yarn test:all           # typecheck + eslint + prettier + test:app --watch=false
yarn test:update        # Vitest, snapshot updates (non-watch)
yarn test:typecheck     # TypeScript (tsc)
yarn fix                # Prettier write + ESLint fix
```

After changes that affect snapshots, run `yarn test:update` before commit. Before a PR, `yarn test:all` is the usual gate (see [`techContext.md`](docs/memory/techContext.md) for nuances).

## Architecture

- **Editor model** — Prefer **actions** and **`syncActionResult`** over ad hoc scene mutation. Scene truth lives in **`@excalidraw/element`** (`Scene`, `Store`, `History`). Deeper patterns: [`docs/memory/systemPatterns.md`](docs/memory/systemPatterns.md).
- **Library vs app** — Shared drawing/editor logic ships in packages; hosting, collab transport, and product-only config lean on **`excalidraw-app/`** (and env/build patterns there).
- **Dual bundles** — Library `exports` expose **dev** vs **prod** entries (`dist/dev` vs `dist/prod`); keep that split when touching package entrypoints.
- **Tests** — Vitest across packages and app; aliases mirror monorepo imports.

Long-form diagrams and dependencies: [`docs/technical/architecture.md`](docs/technical/architecture.md). Local setup: [`docs/technical/dev-setup.md`](docs/technical/dev-setup.md).

## Conventions

- **Code style** — Match existing TypeScript/React patterns; run `yarn fix` and ensure `yarn test:typecheck` passes for touched areas. Component and file naming expectations for `packages/`: see [`.cursor/rules/conventions.mdc`](.cursor/rules/conventions.mdc).
- **Documentation** — For `@excalidraw/excalidraw` props/API and embedding, follow [`dev-docs/docs/@excalidraw/excalidraw/`](dev-docs/docs/@excalidraw/excalidraw/) rather than guessing from internals.
- **Changesets** — Keep edits focused; do not refactor unrelated modules. If you touch protected areas (below), follow the approval and verification path there.
- **PRs** — Describe behavior change, link issues if any, and note test commands you ran (`yarn test:all` or narrower targets when appropriate).

## Do-not-touch / constraints

Do **not** change these paths without **explicit written approval**, full understanding of dependents, **`yarn test:all`**, and recorded manual QA (canvas, open/save, actions as relevant):

- `packages/excalidraw/scene/Renderer.ts` — render pipeline  
- `packages/excalidraw/data/restore.ts` — file format compatibility  
- `packages/excalidraw/actions/manager.tsx` — action system  
- `packages/excalidraw/types.ts` — core types  

Full checklist: [`.cursor/rules/do-not-touch.mdc`](.cursor/rules/do-not-touch.mdc). Related agent rules live under [`.cursor/rules/`](.cursor/rules/) (security, testing, dev-docs, architecture).

## Further reading

| Topic | Location |
|-------|----------|
| MDX developer docs (API, contributing, mermaid, codebase) | [`dev-docs/docs/`](dev-docs/docs/) |
| Toolchain, scripts, Docker | [`docs/memory/techContext.md`](docs/memory/techContext.md) |
| Packages, editor patterns, “where to go deeper” | [`docs/memory/systemPatterns.md`](docs/memory/systemPatterns.md) |
