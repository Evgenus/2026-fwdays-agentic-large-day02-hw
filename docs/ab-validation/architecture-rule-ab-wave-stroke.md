# Rule A/B test: `architecture.mdc`

**Rule:** `.cursor/rules/architecture.mdc`  
**Prompt:** add another style of line stroke (wave)

**Baseline:** branch `master`, clean working tree before runs; rule file confirmed on disk.

---

## Batch A — rule ON

**Git (tracked):**

- `git diff --stat` (after run): 8 files changed, 108 insertions(+), 12 deletions(-)

**Files touched:**

| Path | Role |
|------|------|
| `packages/element/src/waveStroke.ts` | **created** (untracked until reverted) |
| `packages/element/src/index.ts` | export new module |
| `packages/element/src/renderElement.ts` | canvas: wave path for line/arrow; skip rough spine when wave |
| `packages/element/src/shape.ts` | rough options: treat `wave` like `solid` |
| `packages/element/src/types.ts` | `StrokeStyle` includes `"wave"` |
| `packages/excalidraw/actions/actionProperties.tsx` | stroke style UI |
| `packages/excalidraw/components/icons.tsx` | `StrokeStyleWaveIcon` |
| `packages/excalidraw/locales/en.json` | `strokeStyle_wave` label |
| `packages/excalidraw/renderer/staticSvgScene.ts` | SVG export path for wave |

**Subagent summary (compressed):**

- Extended `StrokeStyle` with `"wave"` in element types.
- `waveStroke.ts`: sample rough path, arc-length, perpendicular sine offset; SVG `d` + canvas helpers.
- Canvas: for line/arrow with wave, stroke wavy polyline in 2D; arrowheads still Rough.
- Rough treats `wave` like `solid` for spine geometry.
- SVG mirrors canvas via `<path>` with wave `d`.
- Wave not applied when closed line has non-transparent fill (rough polygon).
- UI: fourth stroke option with icon + `en.json`.

**Protected paths:** `Renderer.ts`, `restore.ts`, `manager.tsx`, `packages/excalidraw/types.ts` — not changed.

**Commands (subagent):** `yarn test:typecheck` passed; element tests reported unrelated `localStorage` failures.

---

## Batch B — rule OFF

**Git (tracked):**

- `git diff --stat` (after run, excluding rule rename): 5 files changed, 211 insertions(+), 25 deletions(-)

**Files touched:**

| Path | Role |
|------|------|
| `packages/element/src/waveStroke.ts` | **created** (untracked until reverted) |
| `packages/element/src/shape.ts` | large change: wave for lines/arrows, rects/embeds, diamonds, ellipses; collision on wavy polyline |
| `packages/element/src/types.ts` | `StrokeStyle` includes `"wave"` |
| `packages/excalidraw/actions/actionProperties.tsx` | stroke style UI |
| `packages/excalidraw/components/icons.tsx` | wave icon |
| `packages/excalidraw/locales/en.json` | label |

**Subagent summary (compressed):**

- `waveStroke.ts`: `applyWaveAlongPolyline`, flatten rough curves, `getWavyEllipsePolyline`.
- Wired wave in `shape.ts` across many shape kinds and hit-testing.
- `generateRoughOptions`: wave like solid for dash behavior.
- UI + locale same class of edits as A.
- No edits to `renderElement.ts`, `staticSvgScene.ts`, or `packages/element/src/index.ts`.

**Protected paths:** not changed.

**Commands (subagent):** `yarn test:typecheck` passed; full `test:app` reported broad failures/snapshots (not attributed to this change alone).

**Orchestration note:** For Batch B, `.cursor/rules/architecture.mdc` was temporarily renamed to `.cursor/rules/architecture.mdc.off`; after capture, working tree was reverted and the rule was renamed back. Final repo state matches pre-test (clean).

---

## Delta matrix

| Area | Batch A (rule ON) | Batch B (rule OFF) |
|------|-------------------|---------------------|
| Files changed | 8 tracked + 1 new | 5 tracked + 1 new |
| Lines / scope | ~108 +12 net on tracked; localized render + export | ~211 +25 net; concentrated in `shape.ts` |
| Tests/docs touched | typecheck only (per agent) | typecheck only (per agent) |
| Canvas / SVG | Explicit split: `renderElement.ts` (canvas) + `staticSvgScene.ts` (SVG) | Primarily `shape.ts` / geometry layer; no `staticSvgScene` / `renderElement` edits in reported set |

---

## Files unique to A / unique to B / changed in both

- **Only A:** `packages/element/src/index.ts` (re-export), `packages/element/src/renderElement.ts`, `packages/excalidraw/renderer/staticSvgScene.ts`
- **Only B:** *(no file unique to B that A did not also touch — B omitted the three paths above)*
- **Both (divergent):** `packages/element/src/types.ts`, `packages/element/src/shape.ts`, `packages/excalidraw/actions/actionProperties.tsx`, `packages/excalidraw/components/icons.tsx`, `packages/excalidraw/locales/en.json`, `packages/element/src/waveStroke.ts` (both runs added a module; implementations and call sites differ)

---

## Impact analysis

1. **Enforcement:** With **architecture.mdc** on, the agent aligned with stated boundaries: **canvas 2D** rendering via the existing render path (`renderElement` / static scene), avoided protected files, and kept the wave stroke implementation **next to** the render pipeline rather than folding all geometry into `shape.ts`. The rule’s emphasis on `renderStaticScene` / static canvas painting plausibly steered updates toward **`staticSvgScene.ts`** for export parity.

2. **Quality:** Batch A’s approach is **narrower** (lines/arrows + export hooks) and may miss wave on rectangles/ellipses that Batch B attempted. Batch B is **broader** (more shapes, collision) but skips explicit SVG static-scene wiring reported in A, which risks **export inconsistency** unless `shape.ts` alone covers all export paths. Neither run modified protected `Renderer.ts` / `types.ts` (app-level).

3. **Cost:** Rule ON produced **more files** but **smaller** per-file churn in core geometry (`shape.ts` +9/-? vs +215 lines). Rule OFF concentrated complexity in **`shape.ts`**, which can be harder to review and merge.

4. **Recommendation:** **Keep** `architecture.mdc` for agent tasks that touch rendering: it nudges toward the documented **static render / canvas** split and away from monolithic `shape.ts` growth. Optional rule tweak: one line explicitly naming **`renderElement.ts`** / **`staticSvgScene.ts`** (or “static export”) when adding stroke styles so SVG and canvas stay paired — without duplicating the protected `Renderer.ts` path.

---

## Verdict

- **Rule effect:** **moderate** — clear difference in **where** implementation landed (render/export layers vs. central `shape.ts`) and **scope** (lines/arrows vs. many shape kinds).
- **One-sentence summary:** For “wave stroke,” the architecture rule correlated with an implementation that touched **render and SVG export modules** and a **smaller** `shape.ts` delta, while the same prompt without the rule produced a **heavier `shape.ts`-centric** solution and **no** reported `staticSvgScene` / `renderElement` edits.

---

## Repro checklist (for humans)

1. Clean tree; confirm `.cursor/rules/architecture.mdc` exists.  
2. Run subagent with prompt + rule ON; capture `git diff`.  
3. `git checkout -- .` and remove any new untracked files from the run.  
4. `git mv .cursor/rules/architecture.mdc .cursor/rules/architecture.mdc.off`.  
5. Run **fresh** subagent with same prompt + note rule disabled.  
6. Capture diff; revert; `git mv` rule back.  
7. Compare using this document’s sections.
