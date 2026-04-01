---
name: code-review
description: >-
  Reviews diffs and PRs against this Excalidraw monorepo’s architecture, security,
  testing, conventions, and protected-file policies. Use when the user asks for a code
  review, PR review, diff review, or pre-merge checklist; or when evaluating changes
  under packages/, excalidraw-app/, or examples/.
---

# Code review (Excalidraw monorepo)

## Scope

Assume the **default gate** before merge is `yarn test:all` when behavior is non-trivial (see `AGENTS.md`). Reviews prioritize **correctness**, **policy compliance**, **security boundaries**, and **maintainability**—not style nits ESLint/Prettier already enforce.

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

For full rule text and “how to verify” steps, read [reference.md](reference.md).

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
