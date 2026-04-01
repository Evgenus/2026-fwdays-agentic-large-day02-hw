# Rule A/B test (Cursor rules)

A/B-test a Cursor rule by running the **same prompt** with the rule **enabled** vs **disabled** (rename to `.mdc.off`), capturing file changesets per run, reverting between runs, then comparing impact.

## Inputs (ask if missing)

1. **Rule to test** — Path under `.cursor/rules/`, e.g. `.cursor/rules/element-model.mdc`.
2. **Prompt** — The exact task a subagent must execute (same text in both runs).

Confirm the rule file exists at the repo root before starting.

## Preconditions

- **Clean or known baseline** — Prefer a clean `git status` for the scope you will revert, or note pre-existing changes and exclude them from batch attribution.
- **Orchestrator vs subagent** — The **parent** session performs git restore and rule rename. **Subagents** only run the prompt and report what they changed. Do not mix rename/revert inside the subagent unless the user explicitly wants that.

## Workflow (execute in order)

### 1) Baseline

- Note current branch and `git status --short`.
- Confirm the rule file exists.

### 2) Batch A — rule ON

- Spawn a **fresh subagent** with the full **prompt** unchanged; project rules apply as on disk (including the rule under test).
- Instruct the subagent to **list every file modified/created/deleted** and summarize edits; if read-only, report deliverables only and record “no filesystem changeset.”
- After it finishes, **capture Batch A**: `git diff` / `git diff --name-only` + focused diffs for attributable changes.
- **Revert** Batch A: `git restore .`; for untracked files, list and remove known paths (use `git clean` only with explicit care).

### 3) Disable the rule

Rename so it is no longer `*.mdc`:

```bash
git mv .cursor/rules/<name>.mdc .cursor/rules/<name>.mdc.off
```

(Untracked rule: use `mv` instead of `git mv`.)

### 4) Batch B — rule OFF

- Spawn a **new subagent** (fresh invocation) with the **same prompt** and explicit note that the rule file is **disabled** (`*.mdc.off`) and must not be followed.
- Capture Batch B the same way as A.

### 5) Re-enable the rule

```bash
git mv .cursor/rules/<name>.mdc.off .cursor/rules/<name>.mdc
```

### 6) Comparison

- **Do not** leave the repo with the rule disabled or with A/B changes—final state should match **pre-test** unless the user asked to keep artifacts.
- Present comparison using the template below.

## Subagent briefing (paste and fill)

```markdown
## Task
<paste prompt verbatim>

## Rules
**Batch A:** Project rules under .cursor/rules/ apply as currently on disk, including: <rule path>.

**Batch B:** The rule file <rule path> is DISABLED (renamed to *.mdc.off). Do not follow its guidance. Other project rules may still apply unless the user says otherwise.

## Reporting
After completing the task:
1. List every created/modified/deleted file (repo paths).
2. Summarize what changed and why in 5–10 bullets.
3. If no files were changed, state that explicitly.
```

## Batch record (repeat for A and B)

```markdown
### Batch <A|B> — rule <ON|OFF>

**Git (tracked):**
- `git diff --stat`
- (optional) `git diff` for key files

**Files touched:** (from subagent + git)

**Subagent summary:**
```

## Comparison report template

```markdown
## Rule A/B test: <rule basename>

**Rule:** `<path>`
**Prompt:** (one-line summary or quoted first line)

### Delta matrix

| Area | Batch A (rule ON) | Batch B (rule OFF) |
|------|-------------------|---------------------|
| Files changed | … | … |
| Lines / scope | … | … |
| Tests/docs touched | … | … |

### Files unique to A / unique to B / changed in both

- **Only A:** …
- **Only B:** …
- **Both (divergent):** …

### Impact analysis

1. **Enforcement:** What did the rule cause (extra steps, forbidden paths, required citations)?
2. **Quality:** Better with rule on or off?
3. **Cost:** Verbosity, time, unnecessary work, false constraints.
4. **Recommendation:** Keep, tighten, loosen, or rewrite; one concrete edit if applicable.

### Verdict

- **Rule effect:** strong | moderate | weak | harmful / noisy
- **One-sentence summary:** …
```

## Failure handling

- Batch A fails: revert working tree, **do not** rename the rule; stop or retry A.
- Batch B fails after rename: revert working tree, **restore** the rule name, then report.
- If unsure about `git clean`: ask the user or only restore tracked files and list untracked paths.

## Notes

- Use a **new** Task/subagent invocation for B so Batch A context does not leak into B.
- Analysis-only prompts: compare **outputs** (length, structure, checklists) instead of diffs.

---

**User context:** The user should provide **rule path** and **prompt**. If they only invoked this command, ask for both before starting.
