---
name: requesting-code-review
description: Use when completing tasks, implementing major features, before opening a pull request (gh pr create), or before merging to verify work meets requirements
---

# Requesting code review

Dispatch the **code-reviewer** agent (see `agents/code-reviewer.md` in this plugin) to catch issues before they cascade. Give the reviewer a **crafted brief**—requirements, git range, and what changed—not your full session history. That keeps the review focused on the work product and preserves your context for follow-up work.

**Core principle:** Review early, review often.

## When to request review

**Mandatory:**

- After each task when using **subagent-development** (or equivalent multi-task subagent flow)
- After completing a **major feature**
- **Before opening a pull request**: see [Before opening a pull request](#before-opening-a-pull-request)
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

## Before opening a pull request

1. Commit everything, then dispatch a **strong-tier** subagent (Claude Code: `Agent` with `model: "opus"`) to review the branch against the PR's base. If the project has its own PR-review skill (Draft Showdown: `review-pr --base <branch>`), have the subagent follow it; otherwise brief it from `code-reviewer.md`. Give it the planned PR title and body as the requirements, not your case for the code. It reports back only: no edits, nothing posted.
2. Verify each finding in the code (**receiving-code-review**), then sort it:
   - **Fix**: the default, cheap nits included.
   - **Skip**: wrong, or not worth the churn. One-line reason.
   - **Needs a call**: see below.
3. Apply the fixes, run the project's checks (**verification**), commit. If the fixes changed logic, review once more; stop after two rounds.
4. Report three lists: Fixed, Skipped (so the user can overrule), Needs a call (options + your recommendation). If anything needs a call, wait for the answers before opening the PR; otherwise open it and include the report.

**Needs a call** = a fix a teammate wouldn't make without asking first:

| Trigger | When |
|---------|------|
| Behavior | The fix changes what users see or how the feature works beyond what the PR set out to do, or more than one behavior is reasonable. |
| Scope | Pre-existing issue, another system, a follow-up ticket, or also needed on another branch (release ↔ main). |
| Decision | Contradicts what the user or the spec chose, or questions the PR's whole approach. |
| Can't verify | Still suspected after one dig, or proving the fix needs a device, play mode, or QA. |
| Risk | Touches save data, serialized assets, server contracts, or deterministic sim code; or the fix is bigger than the change it fixes. |

Unsure whether the reviewer is right → needs a call, not a skip.

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
