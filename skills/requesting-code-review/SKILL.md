---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# Requesting Code Review

Run automated code quality review to catch issues before they cascade. The review produces structured findings with severity, file locations, and recommendations — keeping feedback actionable and precise.

**Core principle:** Review early, review often.

## When to Request Review

**Mandatory:**
- After each task in subagent-driven development
- After completing major feature
- Before merge to main

**Optional but valuable:**
- When stuck (fresh perspective)
- Before refactoring (baseline check)
- After fixing complex bug

## How to Request

**1. Get git SHA:**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main for full branch review
```

**2. Choose review method:**

Ask the user which review method to use:

**Option A: codex-cc automated review**
```bash
REPO_ROOT=$(git rev-parse --show-toplevel)
node "${REPO_ROOT}/plugins/codex-plugin-cc/plugins/codex/scripts/codex-companion.mjs" review --wait --base ${BASE_SHA}
```
Always use `--wait` — the controller needs the result to decide next steps.

**Option B: Claude subagent review**
Dispatch the `superpowers:code-reviewer` subagent using the template at `code-reviewer.md`.

**3. Act on results:**

| Review Severity | Action |
|----------------|--------|
| `critical` | **Must fix** immediately |
| `high` | **Must fix** before proceeding |
| `medium` | Note for later, proceed |
| `low` | Note for later, proceed |

- Verdict `approve` → proceed
- Verdict `needs-attention` + critical/high findings → fix issues, re-run review
- Verdict `needs-attention` + only medium/low → note findings, proceed
- Push back if reviewer is wrong (with reasoning)

## Example

```
[Just completed Task 2: Add verification function]

You: Let me request code review before proceeding.

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')

[Run automated review --base ${BASE_SHA}]

Review output:
  Verdict: needs-attention
  Findings:
    [high] Missing progress indicators — recovery.ts:80-95
      Recommendation: Add progress callback or emit events
    [low] Magic number (100) for reporting interval — recovery.ts:45
      Recommendation: Extract to named constant

You: [Fix progress indicators — high severity]
[Re-run review]
Review: Verdict: approve
[Continue to Task 3]
```

## Integration with Workflows

**Subagent-Driven Development:**
- Review after EACH task (via `code-quality-reviewer-prompt.md` in SDD)
- Catch issues before they compound
- Fix before moving to next task

**Executing Plans:**
- Review after each batch (3 tasks)
- Get feedback, apply, continue

**Ad-Hoc Development:**
- Review before merge
- Review when stuck

## Red Flags

**Never:**
- Skip review because "it's simple"
- Ignore Critical issues
- Proceed with unfixed Important issues
- Argue with valid technical feedback

**If reviewer wrong:**
- Push back with technical reasoning
- Show code/tests that prove it works
- Request clarification

See template at: `requesting-code-review/code-reviewer.md`
