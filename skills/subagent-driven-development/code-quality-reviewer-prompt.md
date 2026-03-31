# Code Quality Reviewer Prompt Template

Use this template when running a code quality review within the SDD per-task review loop.

**Purpose:** Verify implementation is well-built (clean, tested, maintainable)

**Only dispatch after spec compliance review passes.**

## Automated Review (Primary)

Run the automated reviewer via Bash. This returns a structured verdict with findings by severity.

```bash
REPO_ROOT=$(git rev-parse --show-toplevel)
node "${REPO_ROOT}/plugins/codex-plugin-cc/plugins/codex/scripts/codex-companion.mjs" review --wait --base {BASE_SHA}
```

For the final branch review after all tasks:
```bash
node "${REPO_ROOT}/plugins/codex-plugin-cc/plugins/codex/scripts/codex-companion.mjs" review --wait --base origin/main
```

Always use `--wait` — the controller needs the result to decide next steps.

### Interpreting Results

**Approved (proceed):**
- Verdict is `approve`, OR
- Verdict is `needs-attention` but all findings are `medium`/`low` only
- Log medium/low findings as notes, then mark task complete

**Issues found (enter fix loop):**
- Verdict is `needs-attention` AND at least one `critical` or `high` finding
- Dispatch implementer subagent to fix (format below)
- After fixes committed, re-run the same command
- Repeat until approved

### Severity Mapping

| Review Severity | Action |
|----------------|--------|
| `critical` | **Must fix** before proceeding |
| `high` | **Must fix** before proceeding |
| `medium` | Note for later, proceed |
| `low` | Note for later, proceed |

### Dispatching Fixes to Implementer

Provide only the actionable findings:

```
Fix the following code review findings:

1. [{severity}] {title} — {file}:{line_start}-{line_end}
   {body}
   Recommendation: {recommendation}

After fixing, commit your changes.
```

## Claude Subagent Review (Fallback)

If the automated review fails (process error, missing binary, auth issue), dispatch the `castlepowers:code-reviewer` subagent instead.

```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from implementer's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment

## Additional Quality Checks

Regardless of which method runs, the controller should also note:
- Does each file have one clear responsibility with a well-defined interface?
- Are units decomposed so they can be understood and tested independently?
- Is the implementation following the file structure from the plan?
- Did this implementation create new files that are already large, or significantly grow existing files? (Don't flag pre-existing file sizes — focus on what this change contributed.)
