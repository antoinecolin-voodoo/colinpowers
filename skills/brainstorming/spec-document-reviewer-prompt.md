# Spec Document Reviewer Prompt Template

Use this template when dispatching a spec document reviewer via Cursor’s **Task** tool.

**Purpose:** Verify the spec is complete, consistent, and ready for implementation planning.

**Dispatch after:** The spec document is written under `docs/specs/` (for example `docs/specs/YYYY-MM-DD-<topic>-design.md`).

## Cursor Task tool invocation

- Use the **Task** tool to run a subagent whose sole job is spec review.
- Set the subagent to the **standard** model profile (**claude-4.6-sonnet-medium**) when the tool exposes a model choice.
- Put the filled prompt below in the Task **prompt** field. Replace `[SPEC_FILE_PATH]` with the repository-relative path to the spec file.

**Example Task fields (illustrative):**

- **description:** Short label such as `Review design spec`
- **prompt:** The full block below, with `[SPEC_FILE_PATH]` replaced

```
You are a spec document reviewer. Verify this spec is complete and ready for planning.

**Spec to review:** [SPEC_FILE_PATH]

## What to Check

| Category | What to Look For |
|----------|------------------|
| Completeness | TODOs, placeholders, "TBD", incomplete sections |
| Consistency | Internal contradictions, conflicting requirements |
| Clarity | Requirements ambiguous enough to cause someone to build the wrong thing |
| Scope | Focused enough for a single plan — not covering multiple independent subsystems |
| YAGNI | Unrequested features, over-engineering |

## Calibration

**Only flag issues that would cause real problems during implementation planning.**
A missing section, a contradiction, or a requirement so ambiguous it could be
interpreted two different ways — those are issues. Minor wording improvements,
stylistic preferences, and "sections less detailed than others" are not.

Approve unless there are serious gaps that would lead to a flawed plan.

## Output Format

## Spec Review

**Status:** Approved | Issues Found

**Issues (if any):**
- [Section X]: [specific issue] - [why it matters for planning]

**Recommendations (advisory, do not block approval):**
- [suggestions for improvement]
```

**Reviewer returns:** Status, Issues (if any), Recommendations
