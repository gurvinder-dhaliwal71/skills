---
name: technical-code-review
description: Technical implementation review of changes since a fixed point. Breaks down added/modified files and functions by structure and behavior. Use when the user asks for technical-code-review, a technical walkthrough, implementation breakdown, or file/function review of a branch, PR, or WIP diff.
---

Technical implementation review of the diff between `HEAD` and a fixed point the user supplies.

This is not a standards review and not a spec-compliance review. It explains what changed technically.

## Process

### 1. Pin the fixed point

Whatever the user said is the fixed point — a commit SHA, branch name, tag, `main`, `HEAD~5`, etc. If they didn't specify one, ask for it.

Capture the diff command once: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base). Also note the list of commits via `git log <fixed-point>..HEAD --oneline`.

Before going further, confirm the fixed point resolves (`git rev-parse <fixed-point>`) and the diff is non-empty.

### 2. Review changed files

For each added or modified file, report:

1. **Structure** — functions, types, interfaces, classes, modules, routes, or components added/changed.
2. **Behaviour** — one-line summary of the behavior implemented by those structures.

Do not bloat with low-level implementation details unless asked for a deep-dive.

### 3. Output Format

Use this structure:

```md
## Technical Review

### `<file path>`

**Structure:** ...
**Behaviour:** ...

### `<file path>`

**Structure:** ...
**Behaviour:** ...

## Summary

One-line summary of the implementation shape.
```
