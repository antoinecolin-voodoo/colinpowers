---
name: writing-plans-lean
description: Use when an approved spec is too big to implement inline in one session, or the user asks for an implementation plan. Reference-first plans — shorter, reviewable, no full implementation bodies.
---

# Writing Plans (Lean)

## Overview

Write implementation plans that focus on **what to build and how it connects** rather than paste-ready code. Document what to touch: exact paths, codebase references or contract snippets, verification commands, acceptance checks, and docs to read. Group the work into 2–4 workstreams, each a list of tasks. Apply DRY and YAGNI. Prefer frequent, focused commits.

Assume an executor (human or AI agent) who can read the codebase but needs clear direction on intent, boundaries, and acceptance criteria.

## When to write a plan

Default: **no plan**. Implement from the spec inline. Write a plan only when the work is too big for one session (it would fill the context before it's done, typically multi-day work across many files), or when the user asks for one.

**Announce at start:** "I'm using the writing-plans-lean skill to create the implementation plan."

**Save plans to:** `docs/plans/YYYY-MM-DD-<feature-name>.md`  
(User preferences for plan location override this default.)

## Branch Check

Before writing or committing the plan file, verify your branch state:

1. Run `git branch --show-current`.
2. If you are on `main`, `master`, or `develop` — **stop**. Invoke **git-branch-workflow** (Starting work) to create a sub-feature branch first.
3. If brainstorming already created the branch — confirm it is checked out and proceed.

The plan file and all subsequent commits must land on a working branch, not the default branch.

## Scope Check

If the spec spans multiple independent subsystems, it should already be split in brainstorming. If not, recommend separate plans—one per subsystem. Each plan should deliver working software on its own.

## File Structure

Before tasks, map files to create or modify and each file's responsibility. Lock decomposition here.

- Clear boundaries and interfaces; one main responsibility per file.
- Prefer smaller, focused files; split by responsibility, not by arbitrary layers.
- Files that change together should live together when it fits the repo.
- Follow existing project patterns; avoid drive-by restructures unless a touched file clearly needs splitting.

Workstream and task boundaries should align with this map.

## Workstreams

Group the tasks into **2–4 workstreams**, never more than 4. Each one becomes a Linear sub-issue run by one agent (**subagent-development**), so it must stand on its own:

- **Owns its files.** Two workstreams don't edit the same file; if one must, order them.
- **States its dependencies** on other workstreams, so they run in the right order.
- **Ends verifiable:** its own acceptance checks pass once it's merged.

If the work doesn't split into at least two meaningful workstreams, it probably fits inline. Say so instead of forcing a split.

## Describing Implementation Steps

Each implementation step uses one of two modes. **Never** include full method bodies, boilerplate, or paste-ready implementation code.

### Reference mode (preferred)

Point to an existing file, class, or pattern in the codebase and describe how the new code follows or extends it.

> Create `BarService.cs` following the pattern of `Services/FooService.cs`, adding methods `GetBar(id)` and `CreateBar(dto)` that delegate to `IBarRepository`.

### Contract mode (fallback — no existing pattern)

Include only the **public interface** — signatures, type definitions, key data structures. No method bodies. This minimal snippet becomes the reference the executor works from.

```csharp
public interface IBarService {
    Bar GetBar(int id);
    Bar CreateBar(BarDto dto);
}
```

**When to use which:**

- Existing pattern in the codebase? **Reference mode.** Name the file and describe the delta.
- Greenfield component or new architectural pattern? **Contract mode.** Write the minimal contract snippet.

## Task Granularity

Tasks are a workstream's checklist, not units of dispatch: size each as one coherent change that ends in a verify and a commit. Do not use a test-first loop. A typical task uses this rhythm:

1. **Implement** — describe the change with a **codebase reference** or **contract snippet** (see above).
2. **Verify** — compile or run the project's standard check; confirm behavior against acceptance criteria; note expected outcomes.
3. **Commit** — `git add` scope and message; include **Linear issue ID** in the message when one exists (e.g. `feat(scope): summary [ABC-123]`).

Repeat Implement → Verify → Commit per logical slice. Multiple implement sub-steps in one task are fine if each remains small and the verify step covers them together.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** Run inline with **executing-plans** if the plan fits one session; otherwise use **subagent-development** (one agent per workstream). Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence]

**Architecture:** [2–3 sentences]

**Tech Stack:** [Key libraries/tools]

---
```

## Workstream and Task Structure

Each task uses **one** mode — the template below shows both for reference:


````markdown
## Workstream A: [Name]

**Sub-issue:** [ID, added once created] · **Depends on:** [— or Workstream X]
**Acceptance:** [checks that prove this workstream works once merged]

### Task A1: [Component Name]

**Files:**
- Create: `exact/path/to/file.ext`
- Modify: `exact/path/to/existing.ext` (optional: line range)

- [ ] **Step 1: Implement [specific change]**

  **Reference:** Follow the pattern in `path/to/Reference.cs` (lines N–M).
  Create `NewFile.cs` with: [description of responsibilities, public methods, dependencies].

  *Or, if no reference exists:*

  ```language
  // contract only — signatures, types, structures; no method bodies
  public interface INewService {
      Result DoSomething(Input input);
  }
  ```

- [ ] **Step 2: Verify**

  Run: `<project build or check command>`
  Expected: builds cleanly; [specific acceptance checks from spec]

- [ ] **Step 3: Commit**

  ```bash
  git add path/to/touched/files
  git commit -m "type(scope): concise summary [LINEAR-KEY]"
  ```
````

If no Linear issue exists yet, omit the trailing ` [ISSUE-KEY]` from the commit message example in that task; add it in a follow-up commit once linked.

## Quality Bar for Plan Content

- Exact file paths always.
- **Codebase references or contract snippets** — never full implementation bodies, never vague prose. Every step must be precise enough that the executor knows what to create and how it connects, without guessing intent.
- Verification: real commands and what "good" looks like (compile success, manual check, or automated check—whatever the repo uses).
- Reference other colinpowers skills with `@` in chat when helpful (`@writing-plans-lean`, `@executing-plans`, `@subagent-development`, `@linear-integration`).
- DRY, YAGNI, frequent commits.

## Plan Review Loop

After the plan is complete:

1. Dispatch a **plan-document-reviewer** subagent via your platform's subagent tool (Claude Code `Agent` or Cursor `Task`). Give **only** crafted context: path to the plan file, path to the spec/design doc, and review criteria (scope, workstream grouping, paths, reference/contract precision, verify steps, commit hygiene, Linear ID usage). Do not paste full session history.
2. If the reviewer reports issues: fix the plan document, then re-dispatch the reviewer on the **whole** plan.
3. If approved: proceed to execution handoff.

**Guidance:** The same agent that wrote the plan applies fixes. If the loop exceeds **3** iterations, stop and ask a human. Reviewer output is advisory—document respectful disagreement if you reject a point.

## Execution Handoff

After saving the plan, recommend one path and say why:

- **Inline** (**executing-plans**): the whole plan fits in this session.
- **Workstream agents** (**subagent-development**): it doesn't. That skill creates the Linear sub-issues first, then runs one agent per workstream.

## Remember

- Scope check and file map before tasks.
- No plan unless the work won't fit one session.
- 2–4 workstreams, each a sub-issue for one agent; tasks are its checklist. No mandated test-first workflow.
- Implement → verify → commit per task pattern.
- **Reference mode first; contract mode when no pattern exists.**
- Linear IDs in commit messages when available.
- Plan review via **Task** + standard-model reviewer subagent; execution inline (**executing-plans**) or per workstream (**subagent-development**).
