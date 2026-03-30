---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing plans

## Overview

Load the plan, review it critically, execute tasks in order, then close out with the git branch workflow.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Subagents:** When Cursor exposes **Task** (subagent types), use **`subagent-development`** for large plans or clearly independent workstreams instead of doing everything in one thread—throughput and focus usually improve.

## The process

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

1. Announce: "I'm using **git-branch-workflow** (finishing phase) to close out this work."
2. Apply **`git-branch-workflow`** — section **Finishing work**: spec cleanup, squash, Linear update if your project uses it, merge vs PR to the **parent** branch, then checkout parent.

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
| **writing-plans** | Creates the structured plan this skill executes. |
| **git-branch-workflow** | **Finishing work** after all tasks: cleanup, squash, merge or PR to parent, branch hygiene. |
| **subagent-development** | When **Task** subagents are available, split or parallelize execution for suitable plans. |
