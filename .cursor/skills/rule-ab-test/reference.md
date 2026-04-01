# Rule A/B test — reference

## Subagent briefing (paste and fill)

Use two separate invocations with the same skeleton; only the “Rules” paragraph differs.

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

## Batch record template

Repeat for Batch A and Batch B.

```markdown
### Batch <A|B> — rule <ON|OFF>

**Git (tracked):**
- `git diff --stat`
- (optional) `git diff` for key files only

**Files touched:** (from subagent + git)

**Subagent summary:** (paste or compress)

**Artifacts:** links/paths to any outputs outside git if relevant
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

1. **Enforcement:** What did the rule cause the agent to do (extra steps, forbidden paths, required citations)?
2. **Quality:** Correctness, consistency, alignment with repo conventions—better with rule on or off?
3. **Cost:** Verbosity, time, unnecessary work, or false constraints introduced by the rule.
4. **Recommendation:** Keep, tighten, loosen, or rewrite the rule; one concrete edit suggestion if applicable.

### Verdict

- **Rule effect:** strong | moderate | weak | harmful / noisy
- **One-sentence summary:** …
```

## Disable / enable commands (examples)

From repo root, tracked rule:

```bash
git mv .cursor/rules/my-rule.mdc .cursor/rules/my-rule.mdc.off
# ... run Batch B ...
git mv .cursor/rules/my-rule.mdc.off .cursor/rules/my-rule.mdc
```

Untracked rule: use `mv` instead of `git mv`.
