# Code quality reviewer prompt template

Use this template when dispatching a **code quality** reviewer subagent.

**Purpose:** Assess whether the implementation is well built: clear structure, appropriate decomposition, maintainability, and consistency with project norms.

**Only dispatch after spec compliance review has passed.**

**Model:** Use the **standard** tier by default. Escalate to **strong** for large diffs, security-sensitive code, or when a first pass is inconclusive. See `./SKILL.md` (and the authoritative per-platform table in `using-workflow/SKILL.md`).

## Template

Structure the Task prompt so the subagent follows the same shape as **`requesting-code-review/code-reviewer.md`** (relative to the colinpowers skills tree). Use that template’s sections and severity rubric (Critical / Important / Minor) for **Strengths**, **Issues**, and **Assessment**, adapted to the subagent’s read-only review role.

```
Task:
  description: "Code quality review for Task N"
  subagent_type: code-reviewer
  prompt: |
    You are performing a code quality review. Follow the structure and expectations
    defined in requesting-code-review/code-reviewer.md (strengths, categorized issues, assessment).

    ## What was implemented

    [From the implementer's report: summary, files touched, key decisions]

    ## Plan / requirements anchor

    Task N from [plan identifier or title]: [short summary or paste of task scope]

    ## Git range (if available)

    BASE_SHA: [commit before this task]
    HEAD_SHA: [commit after implementer's work]

    Use the diff between BASE and HEAD as primary evidence. If SHAs are unavailable,
    review the files the implementer listed as changed.

    ## Scope

    Focus on quality of **this change**: readability, structure, consistency with nearby
    code, error handling where relevant, and obvious performance or safety smells.

    ## File responsibility checks (in addition to the code-reviewer template)

    In your review, explicitly consider:
    - Does each **new or heavily edited** file have one clear responsibility and a
      coherent interface?
    - Are units decomposed so they can be understood (and, where the project expects it,
      validated) without unnecessary coupling?
    - Does the change respect the **file / module layout** implied by the plan?
    - Did **this task** introduce outsized new files or large growth in existing files?
      (Do not penalize pre-existing file size—only what this change added.)

    ## Output

    Return:
    - **Strengths**
    - **Issues** grouped by Critical / Important / Minor (with file references)
    - **Assessment:** Approved | Approved with minor follow-ups | Not approved (explain)

    If nothing is wrong, say so clearly. Do not invent problems.
```

## Controller notes

- Fill `BASE_SHA` / `HEAD_SHA` when the repo has discrete commits per task; otherwise name paths and rely on the implementer’s file list.
- If the standard review is shallow or contradictory, re-run with **strong** model or a narrower prompt (e.g. single subsystem).
- The implementer addresses findings; you re-dispatch this reviewer until **Approved** (or equivalent) before marking the task done.
