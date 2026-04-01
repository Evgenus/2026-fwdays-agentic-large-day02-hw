---
name: create-rule
description: >-
  Create Cursor rules in .cursor/rules/. Use when adding a new rule, coding
  standards, project conventions, file-specific patterns, or questions about
  rule structure and .cursor/rules/.
---

# Skill: Create Cursor rule

## When to use

- Adding or rewriting a project rule under `.cursor/rules/`.
- User asks for `.mdc` rule structure, `globs`, `alwaysApply`, or verification sections.

## Template file

Copy **[`template.mdc`](template.mdc)** into `.cursor/rules/<descriptive-name>.mdc`, then replace placeholder title, context, bullets, and verification steps. Adjust or remove `globs` and `alwaysApply` per scope below.

## Gather requirements first

1. **Purpose** — What should this rule enforce or teach?
2. **Scope** — Always on, or only when certain files are in context?
3. **Patterns** — If file-specific, the smallest useful glob (e.g. `packages/foo/**/*.ts`).

If scope is unclear, ask whether the rule should always apply or only for specific paths/patterns.

## Frontmatter

| Field | Use |
|--------|-----|
| `description` | One line: what the rule covers and when it applies (shown in rule UI). |
| `globs` | Optional. Quoted string; rule attaches when matching files are relevant. Omit only if relying solely on `alwaysApply`. |
| `alwaysApply` | `true` only when the rule must run outside matched globs; default `false` for scoped rules. |

Narrow `globs` to the smallest useful scope. Do not set `alwaysApply: true` unless the content truly applies everywhere.

## Body structure (match `template.mdc`)

- **`# H1`** — Rule name in plain language (not generic labels like “Rule 1”).
- **`## Context`** — 2–4 sentences: why the rule exists, what problem or risk it addresses.
- **`## Rule`** — Bullet lists; imperative verbs; **NEVER** / **DO NOT** / **AVOID** for prohibitions.
- **`## How to verify`** — Numbered, actionable steps (commands, files to open, checks) someone can run in this repo.

Keep rules concise; one concern per file; prefer concrete examples when they clarify behavior.

## After you write the file

1. Confirm YAML frontmatter parses (`description`, optional quoted `globs`, boolean `alwaysApply`).
2. If `alwaysApply` is `false`, confirm `globs` match the intended files and do not over-capture.
3. Read the rule end-to-end: every bullet actionable; “How to verify” steps are real checks for this project.

## Outputs

- New or updated `.cursor/rules/<name>.mdc` derived from [`template.mdc`](template.mdc).
