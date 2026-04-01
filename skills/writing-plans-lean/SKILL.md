---
name: writing-plans-lean
description: Use when you have a spec or requirements for a multi-step task, before touching code. Reference-first plans — shorter, reviewable, no full implementation bodies.
---

# Writing Plans (Lean)

## Overview

Write implementation plans that focus on **what to build and how it connects** rather than paste-ready code. Document what to touch: exact paths, codebase references or contract snippets, verification commands, acceptance checks, and docs to read. Decompose into bite-sized tasks. Apply DRY and YAGNI. Prefer frequent, focused commits.

Assume an executor (human or AI agent) who can read the codebase but needs clear direction on intent, boundaries, and acceptance criteria.

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

Task boundaries should align with this map.

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

## Bite-Sized Task Granularity

**Each step is one short action (about 2–5 minutes).** Do not use a test-first loop. A typical task uses this rhythm:

1. **Implement** — describe the change with a **codebase reference** or **contract snippet** (see above).
2. **Verify** — compile or run the project's standard check; confirm behavior against acceptance criteria; note expected outcomes.
3. **Commit** — `git add` scope and message; include **Linear issue ID** in the message when one exists (e.g. `feat(scope): summary [ABC-123]`).

Repeat Implement → Verify → Commit per logical slice. Multiple implement sub-steps in one task are fine if each remains small and the verify step covers them together.

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** Use **subagent-development** (recommended) or **executing-plans** to run this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence]

**Architecture:** [2–3 sentences]

**Tech Stack:** [Key libraries/tools]

---
```

## Task Structure

Each task uses **one** mode — the template below shows both for reference:


````markdown
### Task N: [Component Name]

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

1. Use Cursor's **Task** tool to dispatch a **plan-document-reviewer** subagent with a **standard** model profile. Give **only** crafted context: path to the plan file, path to the spec/design doc, and review criteria (scope, paths, reference/contract precision, verify steps, commit hygiene, Linear ID usage). Do not paste full session history.
2. If the reviewer reports issues: fix the plan document, then **Task** the reviewer again on the **whole** plan.
3. If approved: proceed to execution handoff.

**Guidance:** The same agent that wrote the plan applies fixes. If the loop exceeds **3** iterations, stop and ask a human. Reviewer output is advisory—document respectful disagreement if you reject a point.

## Execution Handoff

After saving the plan, offer:

**"Plan saved to `docs/plans/<filename>.md`. Two execution options:**

**1. Subagent-driven (recommended)** — **Task** dispatches a fresh subagent per task; review between tasks; fast iteration. Follow **subagent-development**.

**2. Inline execution** — Run tasks in this session with checkpoints for review. Follow **executing-plans**.

**Which do you want?"**

**If subagent-driven:** require **subagent-development**; one subagent per plan task (or per batch if that skill defines batching).

**If inline:** require **executing-plans**; batch with explicit review checkpoints between groups of tasks.

## See Also

- **writing-plans** — the original variant with complete paste-ready code in the plan. Use when explicitly requested or when the executor has no codebase access.

## Remember

- Scope check and file map before tasks.
- Bite-sized steps without a mandated test-first workflow.
- Implement → verify → commit per task pattern.
- **Reference mode first; contract mode when no pattern exists.**
- Linear IDs in commit messages when available.
- Plan review via **Task** + standard-model reviewer subagent; execution via **subagent-development** or **executing-plans**.
