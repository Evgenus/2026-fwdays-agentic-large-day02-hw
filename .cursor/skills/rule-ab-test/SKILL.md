---
name: rule-ab-test
description: >-
  A/B-tests a Cursor rule by running the same prompt with the rule enabled vs
  disabled (rename to .mdc.off), capturing file changesets per run, reverting
  between runs, then comparing impact. Use when the user wants to measure how a
  rule affects agent outputs, evaluate rule effectiveness, or compare behavior
  with/without a specific .cursor/rules entry.
---

# Rule A/B test (Cursor rules)

## Inputs (required)

1. **Rule to test** — Path to the rule file, usually under `.cursor/rules/`, e.g. `.cursor/rules/element-model.mdc`.
2. **Prompt** — The exact task the subagent must execute (same text in both runs).

Resolve the rule path from the repo root; confirm the file exists before starting.

## Outcomes to produce

- **Batch A** — Changeset from the prompt with the rule **enabled** (current state).
- **Batch B** — Changeset from the same prompt with the rule **disabled**.
- **Comparison** — Structured analysis of how the rule affected files touched, patterns, and quality (see [reference.md](reference.md)).

## Preconditions

- **Clean or known baseline** — Prefer a clean `git status` for the scope you will revert, or note pre-existing changes and exclude them from “batch” attribution.
- **Orchestrator vs subagent** — The **parent** session performs git restore and rule rename. **Subagents** only run the prompt and report what they changed (or produce a patch summary). Do not mix rename/revert inside the subagent unless the user explicitly wants that.

## Workflow

Execute in order. Do not skip revert or restore steps.

### 1) Baseline

- Note current branch and `git status --short` (or equivalent) for the repo root.
- Confirm the rule file exists at the given path.

### 2) Batch A — rule ON

- Spawn a **subagent** (e.g. Task tool, `generalPurpose` or `explore` as fits the prompt) with:
  - The full **prompt** unchanged.
  - Instruction that **all project Cursor rules apply** as on disk (including the rule under test).
  - Instruction to **list every file modified/created/deleted** and summarize edits; if the task is read-only, report deliverables only and record “no filesystem changeset.”
- After the subagent finishes, **capture Batch A**:
  - `git diff` (or `git diff --name-only` + focused diffs) for all changes attributable to the run.
  - Optional: short bullet summary of behavioral differences you observe in the output (structure, citations, checks performed).
- **Revert** all working tree changes from Batch A: `git restore .` and remove untracked files the run created if safe (`git clean` only with explicit care—prefer listing and deleting known paths). Goal: workspace matches pre-A state except you will temporarily rename the rule next.

### 3) Disable the rule

- **Turn off** the rule by appending **`.off`** to the filename so it is no longer a `*.mdc` rule, e.g.  
  `element-model.mdc` → `element-model.mdc.off`  
  Use `git mv` if the rule is tracked, so enable/disable stays reviewable.

### 4) Batch B — rule OFF

- Spawn a **fresh subagent** with:
  - The **same prompt** as Batch A.
  - Explicit note that the rule file under test is **disabled on disk** (`*.mdc.off`) and must not be treated as active project guidance.
- **Capture Batch B** the same way as Batch A (diff + summary).

### 5) Re-enable the rule

- Rename back: `element-model.mdc.off` → `element-model.mdc` (use `git mv` if applicable).
- Confirm the rule path is restored.

### 6) Comparison analysis

- **Do not** leave the repo with the rule disabled or with Batch A/B changes applied—final state should match **pre-test** except any intentional artifacts the user asked to keep.
- Present the comparison using the template in [reference.md](reference.md): files only in A, only in B, both with diff highlights; qualitative impact of the rule (enforcements followed, extra steps, scope creep, noise).

## Failure handling

- If Batch A fails: revert working tree, **do not** rename the rule; stop or retry A.
- If Batch B fails after rename: revert working tree, **restore** the rule name, then report.
- If unsure about `git clean`: ask the user or only restore tracked files and list untracked paths for manual removal.

## Notes

- **Subagent freshness** matters: use a **new** Task invocation for B so prior conversation does not leak “memory” of Batch A outputs into B’s reasoning when comparing behavior.
- Rules with `alwaysApply` or broad `globs` can still affect other agents in parallel—avoid running other rule-changing operations during the test.
- For prompts that **should not** modify the repo (analysis-only), batches are **output-only**; compare deliverables (length, structure, checklists) instead of diffs.

## Additional resources

- Batch logging and comparison template: [reference.md](reference.md)
