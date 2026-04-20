---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# Requesting code review

Dispatch the **code-reviewer** agent (see `agents/code-reviewer.md` in this plugin) to catch issues before they cascade. Give the reviewer a **crafted brief**—requirements, git range, and what changed—not your full session history. That keeps the review focused on the work product and preserves your context for follow-up work.

**Core principle:** Review early, review often.

## When to request review

**Mandatory:**

- After each task when using **subagent-development** (or equivalent multi-task subagent flow)
- After completing a **major feature**
- **Before merge** to the main integration branch

**Optional but valuable:**

- When stuck (fresh perspective)
- Before a large **refactoring** (baseline check)
- After fixing a **complex bug**

## How to request

**1. Get git SHAs**

Use a base that matches the scope of the change (parent of your work, `origin/main`, or the commit before your series):

```bash
BASE_SHA=$(git rev-parse origin/main)   # or HEAD~N / a known base commit
HEAD_SHA=$(git rev-parse HEAD)
```

**2. Dispatch via your subagent tool**

- Claude Code: `Agent` tool with `subagent_type: "code-reviewer"` (see `agents/code-reviewer.md`).
- Cursor: `Task` tool targeting the same `agents/code-reviewer.md` agent definition.
- Build the subagent **prompt** from the template in **`skills/requesting-code-review/code-reviewer.md`**: substitute every `{…}` placeholder with concrete text and SHAs (see that file for the full structure).

**3. Act on feedback**

- Fix **Critical** issues immediately.
- Fix **Important** issues before proceeding or merging.
- Defer **Minor** issues only when explicitly tracked.
- Push back if the reviewer is wrong—cite code, behavior, or requirements.

## Example workflow

You finished a task and want review before the next one.

1. Choose `BASE_SHA` / `HEAD_SHA` for the commits that contain this task’s changes.
2. Open **Task** → **code-reviewer** agent (`agents/code-reviewer.md`).
3. Paste a filled-in brief from `code-reviewer.md` (implementation summary, plan/requirements, SHAs, short description).
4. Read the verdict. If **Important** items appear, fix them and optionally re-run review on the new range.

## Integration with workflows

| Workflow | How review fits in |
|----------|-------------------|
| **subagent-development** | After **each** task: review before starting the next so defects do not stack. |
| **executing-plans** | After each batch or milestone: review, apply feedback, then continue the plan. |
| **Ad-hoc** | At minimum before merge; also when stuck or after risky changes. |

## Red flags

**Never:**

- Skip review because the change “looks small”
- Ignore **Critical** issues
- Merge or continue the next major step with unfixed **Important** issues
- Dismiss valid technical feedback without reasoning

**If the reviewer is wrong:** Respond with technical evidence (code paths, requirements, observed behavior) and ask for a narrowed re-check if needed.

**Template for the subagent brief:** `skills/requesting-code-review/code-reviewer.md`
