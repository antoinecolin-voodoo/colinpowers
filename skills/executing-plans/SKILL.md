---
name: executing-plans
description: Use when executing a written implementation plan inline, in this session or a new one, with review checkpoints
---

# Executing plans

## Overview

Load the plan, review it critically, execute tasks in order, then close out with the git branch workflow.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Too big for one session?** Use **subagent-development** instead: one agent per workstream, each on its own Linear sub-issue.

## The process

### Step 0: Branch check

Before executing any code changes, verify your branch state:

1. Run `git branch --show-current`.
2. If you are on `main`, `master`, or `develop` — **stop**. Invoke **git-branch-workflow** (Starting work) to create a sub-feature branch before writing any code.
3. If the current branch is a feature branch but this plan warrants its own sub-feature branch — invoke **git-branch-workflow** (Starting work).
4. If brainstorming already created the branch — confirm it is checked out and proceed.

Do not skip this step even if the plan does not mention branching.

### Step 1: Load and review plan

1. Read the plan file (or the plan the user pasted).
2. Review critically: gaps, unclear steps, risky assumptions, missing verifications.
3. If you have concerns, raise them with the user before writing code.
4. If the plan is acceptable, create a **TodoWrite** list that mirrors the plan’s tasks and proceed.

### Step 2: Execute tasks

For each task:

1. Mark it **in_progress** in **TodoWrite**.
2. Follow each step as written (plans should keep steps small and explicit).
3. Run every verification the plan specifies (build, play mode, scripts, linters—whatever it lists).
4. Mark **completed** when the task and its checks are done.

### Step 3: Finish development

After all tasks are complete and verified:

1. **Unity validation gate** — follow **git-branch-workflow** § **Finishing work → 0. Unity validation gate**: `unity status` for an Editor already holding the worktree, then `unity cmd menu --path "Assets/Refresh"` and poll `unity cmd recompile_status` until `completed`/`up_to_date` with no `errors[]`; with no Editor open, `unity open .` (poll until `ready`) or the headless `unity test . --mode EditMode` gate at exit `0`. If the Unity CLI is unavailable, the run fails, or **visual / scene-facing** work still needs human judgment, ask the user to validate in Unity. Commit `.meta` (and related import) changes after the gate passes, leaving pipeline-dirtied project settings out.
2. Announce: "I'm using **git-branch-workflow** (finishing phase) to close out this work."
3. Apply **`git-branch-workflow`** — section **Finishing work** (from step 1 onward): spec cleanup, squash, Linear update if your project uses it, merge vs PR to the **parent** branch, then checkout parent.

## When to stop and ask

**Stop executing immediately when:**

- You hit a blocker (missing dependency, verification keeps failing, instruction is ambiguous).
- The plan has critical gaps that prevent a safe start.
- You do not understand a step.

**Ask for clarification instead of guessing.**

## When to revisit Step 1

Return to load-and-review when:

- The user revises the plan from your feedback.
- The fundamental approach needs rethinking.

Do not push through blockers silently.

## Remember

- Review the plan critically before coding.
- Follow plan steps and verifications; do not skip checks the plan requires.
- Use other skills when the plan or workflow tells you to.
- Stop when blocked; do not guess.
- Do not start implementation on **main** / **master** without explicit user consent.

## Integration

| Skill | Role |
|--------|------|
| **writing-plans-lean** | Creates the structured plan this skill executes. |
| **git-branch-workflow** | **Finishing work** after all tasks: cleanup, squash, merge or PR to parent, branch hygiene. |
| **subagent-development** | When the plan doesn't fit one session: one agent per workstream. |
