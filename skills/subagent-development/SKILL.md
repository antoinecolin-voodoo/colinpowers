---
name: subagent-development
description: Use when an implementation plan is too big to execute inline in one session and needs to be split across agents
---

# Subagent development

## Overview

Run a big plan with **one agent per workstream**: at most 4, usually 2–3. Each workstream first becomes a Linear sub-issue with its own sub-feature branch, so each agent owns one issue end to end. You coordinate: brief, review, merge.

Agents get big slices, not single tasks. A fresh context per slice is what makes a big plan tractable; per-task agents spend more on briefing and review than on the work.

## When to use

- A **writing-plans-lean** plan that won't fit in this session. If it fits, run it inline with **executing-plans**.
- Not for unplanned parallel fixes: see **dispatching-parallel**.

## The process

1. **Group.** Check the plan's workstreams: 2–4, each owning its files, listing what it depends on, and ending in its own acceptance checks. If the plan is still written task by task, group it now by merging tasks until each group is a slice one agent can finish end to end. Never more than 4. If you can't find 2 meaningful groups, it's inline work.
2. **Sub-issues.** Follow **linear-integration** (Sub-issues): show the user the list, create after their go-ahead, then write each ID into its workstream header and commit the plan.
3. **Run each workstream**, in dependency order:
   1. **git-branch-workflow** (Starting work) off the parent branch; set the sub-issue In Progress.
   2. Dispatch one implementer from `./implementer-prompt.md`.
   3. On DONE: review the sub-branch with **requesting-code-review**, using the sub-issue's acceptance checks as the requirements, and triage with its fix / skip / needs-a-call rule. Send fixes back to the same implementer (Claude Code: `SendMessage`); bring the calls to the user.
   4. **git-branch-workflow** (Finishing work) for the sub-branch: gate, one commit `<SUB-ID>: …`, merge into the parent, sub-issue to its handoff state.
4. **Finish the parent** with **git-branch-workflow** (Finishing work) once every workstream is merged, including the pre-PR review if it ends in a PR.

**Parallel when possible.** Two workstreams that share no files and don't depend on each other can run at the same time: propose it. **Never create a worktree** (`Agent` with `isolation: "worktree"` included). Ask the user which of their existing worktrees the second agent may use; check it's clean, then create the sub-branch there (`git -C <worktree> switch -c <sub-branch> <parent>`). When that workstream is done, finish it there (gate, squash), switch the worktree back to its original branch, then merge from this checkout.

Everything else runs one agent at a time in this checkout, each starting from a parent that already holds the earlier workstreams.

## Briefing the implementer

Paste the workstream's section of the plan, plus the sub-issue ID, its branch, the spec path, and what the earlier workstreams delivered (new APIs, files, behavior). The agent reads the spec and the code; it doesn't need the rest of the plan.

## Implementer status

- **DONE**: review it.
- **DONE_WITH_CONCERNS**: read the concerns first; settle the ones that touch correctness or scope before review.
- **NEEDS_CONTEXT**: supply it; the same agent continues.
- **BLOCKED**: change something: more context, a smaller slice, or the user if the plan is wrong. Never resend the same brief unchanged.

## Red flags

- One agent per task, or more than 4 agents.
- Creating sub-issues before the user OKs the list.
- Two agents in the same checkout or on the same branch.
- Creating a worktree, or using an existing one without the user's OK.
- Starting a workstream before the ones it depends on are merged into the parent.
- Merging a sub-branch with open review findings, or without the Unity validation gate (**git-branch-workflow** §0).
- Rewriting an implementer's work yourself: send it back with explicit instructions, unless the fix is a line or two.

## Integration

| Skill | Role |
|-------|------|
| **writing-plans-lean** | Writes the plan and its workstreams. |
| **linear-integration** | Creates the sub-issues; handoff states. |
| **requesting-code-review** | Review per workstream, then the pre-PR review. |
| **git-branch-workflow** | One sub-branch per workstream, then the parent. |
| **executing-plans** | The inline alternative when the plan fits one session. |
