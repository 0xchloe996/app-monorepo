---
name: 1k-code-review-expert
description: "Expert code review of current git changes with a senior engineer lens. Auto-runs after writing code or fixing bugs. Detects SOLID violations, security risks, performance issues, error handling gaps, boundary conditions, and proposes actionable improvements. Code review. Self review. Bug fix review."
allowed-tools: Read, Grep, Glob, Bash
---

# Code Review Expert

**Output language**: Use Chinese for all review report content.

Perform a structured review of the current git changes with focus on SOLID, architecture, removal candidates, security risks, and code quality. Default to review-only output unless the user asks to implement changes.

## When to Use

This skill should be invoked **automatically** after:
- Writing new code or features
- Fixing bugs
- Refactoring code
- Any code change before creating a PR

## Quick Reference

| Topic | Guide | Focus |
|-------|-------|-------|
| SOLID + Architecture | [solid-checklist.md](references/rules/solid-checklist.md) | SRP, OCP, LSP, ISP, DIP violations |
| Security & Reliability | [security-checklist.md](references/rules/security-checklist.md) | XSS, injection, SSRF, race conditions, secrets |
| Code Quality | [code-quality-checklist.md](references/rules/code-quality-checklist.md) | Error handling, performance, boundary conditions |
| Removal Plan | [removal-plan.md](references/rules/removal-plan.md) | Dead code identification, safe deletion |

## Severity Levels

| Level | Name | Description | Action |
|-------|------|-------------|--------|
| **P0** | Critical | Security vulnerability, data loss risk, correctness bug | Must block merge |
| **P1** | High | Logic error, significant SOLID violation, performance regression | Should fix before merge |
| **P2** | Medium | Code smell, maintainability concern, minor SOLID violation | Fix in this PR or create follow-up |
| **P3** | Low | Style, naming, minor suggestion | Optional improvement |

## Workflow

### 1) Preflight Context

- Use `git status -sb`, `git diff --stat`, and `git diff` to scope changes.
- If needed, use `rg` or `grep` to find related modules, usages, and contracts.
- Identify entry points, ownership boundaries, and critical paths (auth, payments, data writes, network).

**Edge cases:**
- **No changes**: If `git diff` is empty, inform user and ask if they want to review staged changes or a specific commit range.
- **Large diff (>500 lines)**: Summarize by file first, then review in batches by module/feature area.
- **Mixed concerns**: Group findings by logical feature, not just file order.

### 2) SOLID + Architecture Smells

See: [references/rules/solid-checklist.md](references/rules/solid-checklist.md)

- **SRP**: Overloaded modules with unrelated responsibilities.
- **OCP**: Frequent edits to add behavior instead of extension points.
- **LSP**: Subclasses that break expectations or require type checks.
- **ISP**: Wide interfaces with unused methods.
- **DIP**: High-level logic tied to low-level implementations.

When proposing a refactor, explain *why* it improves cohesion/coupling and outline a minimal, safe split. If refactor is non-trivial, propose an incremental plan instead of a large rewrite.

### 3) Removal Candidates + Iteration Plan

See: [references/rules/removal-plan.md](references/rules/removal-plan.md)

- Identify code that is unused, redundant, or feature-flagged off.
- Distinguish **safe delete now** vs **defer with plan**.
- Provide a follow-up plan with concrete steps and checkpoints.

### 4) Security and Reliability Scan

See: [references/rules/security-checklist.md](references/rules/security-checklist.md)

- XSS, injection (SQL/NoSQL/command), SSRF, path traversal
- AuthZ/AuthN gaps, missing tenancy checks
- Secret leakage or API keys in logs/env/files
- Rate limits, unbounded loops, CPU/memory hotspots
- Race conditions: concurrent access, check-then-act, TOCTOU, missing locks

Call out both **exploitability** and **impact**.

### 5) Code Quality Scan

See: [references/rules/code-quality-checklist.md](references/rules/code-quality-checklist.md)

- **Error handling**: swallowed exceptions, overly broad catch, missing error handling, async errors
- **Performance**: N+1 queries, CPU-intensive ops in hot paths, missing cache, unbounded memory
- **Boundary conditions**: null/undefined handling, empty collections, numeric boundaries, off-by-one

Flag issues that may cause silent failures or production incidents.

### 6) Output Format

Structure the review as follows:

```markdown
## Code Review Summary

**Files reviewed**: X files, Y lines changed
**Overall assessment**: [APPROVE / REQUEST_CHANGES / COMMENT]

---

## Findings

### P0 - Critical
(none or list)

### P1 - High
1. **[file:line]** Brief title
  - Description of issue
  - Suggested fix

### P2 - Medium
2. (continue numbering across sections)
  - ...

### P3 - Low
...

---

## Removal/Iteration Plan
(if applicable)

## Additional Suggestions
(optional improvements, not blocking)
```

**Clean review**: If no issues found, explicitly state:
- What was checked
- Any areas not covered
- Residual risks or recommended follow-up tests

### 7) Next Steps Confirmation

After presenting findings, ask user how to proceed:

```markdown
---

## Next Steps

I found X issues (P0: _, P1: _, P2: _, P3: _).

**How would you like to proceed?**

1. **Fix all** - I'll implement all suggested fixes
2. **Fix P0/P1 only** - Address critical and high priority issues
3. **Fix specific items** - Tell me which issues to fix
4. **No changes** - Review complete, no implementation needed

Please choose an option or provide specific instructions.
```

**Important**: Do NOT implement any changes until user explicitly confirms. This is a review-first workflow.

## OneKey-Specific Checks

In addition to the general review, pay attention to:

- **Cross-platform impact**: Check if changes affect extension/mobile/desktop/web
- **i18n**: Ensure new user-facing strings use `ETranslations`
- **Import hierarchy**: Verify `apps/*` only imports from `packages/*`, not other apps
- **Comments in English**: All code comments must be in English
- **Jotai state patterns**: Verify correct atom usage (globalAtom, contextAtom)

## Related Skills

- `/1k-code-review-pr` - PR-level code review (build reliability, runtime quality)
- `/pr-review` - Security-first PR review (secrets, supply-chain, cross-platform)
- `/1k-code-quality` - Lint, type check, pre-commit hooks
- `/1k-coding-patterns` - React components, TypeScript conventions
- `/1k-performance` - Re-renders, memoization, memory leaks
- `/1k-error-handling` - Try/catch, error boundaries, toast messages
- `/1k-architecture` - Project structure, import hierarchy rules
