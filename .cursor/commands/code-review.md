# Code review (Excalidraw monorepo)

You are reviewing a diff, PR, or change set for this repo. Follow this workflow and output format.

## Scope

Assume the **default gate** before merge is `yarn test:all` when behavior is non-trivial (see `AGENTS.md`). Prioritize **correctness**, **policy compliance**, **security boundaries**, and **maintainability**—not style nits ESLint/Prettier already enforce.

## Workflow

1. **Identify the change set** — files touched, packages (`packages/excalidraw`, `packages/element`, `excalidraw-app`, etc.), and whether the diff is library vs app vs examples.
2. **Protected files** — If any of these appear in the diff, flag **Must fix / policy**: explicit written approval path, `yarn test:all`, and recorded manual QA are required before merge:
   - `packages/excalidraw/scene/Renderer.ts`
   - `packages/excalidraw/data/restore.ts`
   - `packages/excalidraw/actions/manager.tsx`
   - `packages/excalidraw/types.ts`
   Suggest verifying with: `git diff --name-only <merge-base>...HEAD` (or equivalent) against those paths.
3. **Architecture fit** — `packages/excalidraw`: state through **ActionManager** / **`syncActionResult`**, not ad hoc global mutation; canvas **2D** for drawing, not React DOM / Konva / Fabric / Pixi for the scene. **No new npm deps** without documented approval; prefer `packages/utils` and existing workspace code.
4. **`packages/element`** — Scene/Store/mutateElement bookkeeping; no ActionManager or React UI in this package; respect undo/capture semantics; avoid upward runtime cycles.
5. **Security** — Embeds/links: keep validation before render/nav (`validateEmbeddable`, `embeddableURLValidator`, hyperlink paths). No weakening default URL/embed rules without a stated threat model. Untrusted JSON/library/files: no expanded unsafe eval/deserialization. Collab/auth in **excalidraw-app**, not assumed in the library. No secrets in source; prod paths must not ship dev-only logging or permissive CORS by mistake.
6. **Conventions** — Functional components + hooks; **named exports** (no default exports for components in `packages/`); props type `*Props`; kebab-case non-component files, PascalCase component files; strict TS: no `any`, no `@ts-ignore` in new or heavily touched code; `import type` for types.
7. **Tests** — Vitest + Testing Library; colocate `ComponentName.test.tsx` where applicable; reuse `packages/excalidraw/tests/helpers` when bootstrapping editor tests; snapshot updates only when intentional (`yarn test:update`). Call out missing coverage for bug fixes or risky branches.
8. **Docs / API claims** — If the change affects public embedding API or integration behavior, note whether `dev-docs/docs/` (especially `@excalidraw/excalidraw/`) should be updated—not every internal change needs MDX.

### Authoritative sources (read when exact wording or verification steps matter)

| Topic | Path |
|-------|------|
| Agent overview, commands, do-not-touch list | `AGENTS.md` |
| Protected files policy | `.cursor/rules/do-not-touch.mdc` |
| Architecture (state, rendering, deps) | `.cursor/rules/architecture.mdc` |
| TS/React/file naming | `.cursor/rules/conventions.mdc` |
| Security (embeds, URLs, scene data, collab, secrets) | `.cursor/rules/security.mdc` |
| Vitest, snapshots, PR test expectations | `.cursor/rules/testing.mdc` |
| When to use `dev-docs/docs/` | `.cursor/rules/dev-docs.mdc` |
| Scene, Store, element mutations | `.cursor/rules/element-model.mdc` |
| Toolchain detail | `docs/memory/techContext.md` |
| Package boundaries, editor patterns | `docs/memory/systemPatterns.md` |

## Output format

Structure the review as:

### Summary

2–4 sentences: what the change does and overall risk (low/medium/high).

### Must fix

Blocking issues: correctness bugs, policy violations (including protected files without approval path), security regressions, broken types/build/tests.

### Should fix

Important but not always blocking: missing tests for risky logic, unclear boundaries (library vs app), fragile patterns that violate architecture or element-model rules.

### Consider / nit

Optional improvements: naming, small refactors, documentation clarifications.

### Checklist (for the author)

- [ ] `yarn test:typecheck` (and `yarn test:all` or scoped tests as appropriate)
- [ ] No unintended edits to protected files (or approval + QA documented)
- [ ] Security-sensitive paths still validate untrusted input
- [ ] Tests added/updated for behavior changes

Keep feedback **specific**: file paths, symbols, and suggested direction—not vague praise or generic advice.

## Anti-patterns in review

- Do not request large unrelated refactors.
- Do not treat this repo as Redux/Zustand-based; align with ActionManager patterns.
- Do not approve bypassing Scene/Store element updates in `packages/element` without a strong, stated reason.

---

**User context:** Apply the above to the change set the user describes (branch, PR link, pasted diff, or files they name). If nothing is specified, ask what to review or infer from git/selection.
