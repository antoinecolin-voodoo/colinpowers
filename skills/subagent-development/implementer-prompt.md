# Implementer subagent prompt template

Use this template when dispatching an **implementer** subagent via Cursor’s **Task** tool.

**Model:** Use `model: "fast"` on the Task call when the task is mechanical (1–2 files, clear spec). Use your environment’s mapping to **standard** or **strong** when the task needs more judgment (see `SKILL.md`).

```
Task:
  description: "Implement Task N: [short task name]"
  model: "fast"
  subagent_type: generalPurpose
  prompt: |
    You are implementing Task N: [task name]

    ## Task description

    [FULL TEXT of the task from the plan—paste here. Do not rely on reading the plan file yourself unless explicitly told to.]

    ## Context

    [Scene-setting: where this fits, dependencies, constraints, architecture notes.]

    ## Before you begin

    If anything is unclear about:
    - Requirements or acceptance criteria
    - Approach or implementation strategy
    - Dependencies or assumptions
    - Wording or scope in the task

    **Ask now.** Raise concerns before writing code.

    ## Your job

    Once requirements are clear:
    1. Implement exactly what the task specifies.
    2. **Verify** your implementation compiles and meets the acceptance criteria (build, manual checks, or project-standard checks as applicable).
    3. Commit your work.
    4. Self-review (see below).
    5. Report back in the required format.

    Work from: [directory or repo root]

    **While you work:** If something unexpected or ambiguous appears, **stop and ask**. Do not guess or broaden scope silently.

    ## Code organization

    You reason best when files stay focused. Keep this in mind:
    - Follow the file structure defined in the plan.
    - Each file should have one clear responsibility and a well-defined interface.
    - If a new file is growing beyond the plan’s intent, stop and report **DONE_WITH_CONCERNS**—do not split or reorganize without plan guidance.
    - If an existing file you touch is already large or tangled, proceed carefully and note it as a concern.
    - In existing codebases, follow established patterns. Improve only what you touch in reasonable ways; do not restructure unrelated areas.

    ## When you are stuck

    It is always acceptable to say the task is too hard or too unclear. Poor work is worse than none. Escalation is not a failure.

    **Stop and escalate** (BLOCKED or NEEDS_CONTEXT) when:
    - The task needs architectural choices among several valid approaches not decided in the plan.
    - You need understanding beyond what was provided and cannot resolve it by reading the codebase in reasonable time.
    - You are unsure your approach matches intent.
    - The plan did not anticipate required refactors of existing code.
    - You are circling without progress after substantial exploration.

    **How to escalate:** Report **BLOCKED** or **NEEDS_CONTEXT**. State what you tried, exactly what is missing or blocking, and what would unblock you (context, stronger review, smaller task, human decision).

    ## Before reporting back: self-review

    **Completeness**
    - Did you implement everything in the task and acceptance criteria?
    - Any missed requirements or edge cases?

    **Quality**
    - Is this your best work for this scope?
    - Are names accurate and clear?
    - Is the code readable and maintainable?

    **Discipline**
    - Did you avoid overbuilding (YAGNI)?
    - Did you stay within the requested scope?
    - Did you follow existing project patterns?

    **Verification**
    - Does the project build (or the relevant subset)?
    - Do the acceptance criteria you were given actually hold for what you implemented?

    Fix issues you find before reporting.

    ## Report format

    When done, report:
    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - What you implemented (or what you attempted, if blocked)
    - **Verification:** what you ran or checked (compile, criteria, manual steps) and the outcome
    - Files changed (paths)
    - Self-review findings (if any)
    - Issues or concerns

    Use **DONE_WITH_CONCERNS** if you finished but have correctness or scope doubts.
    Use **BLOCKED** if you cannot complete the task.
    Use **NEEDS_CONTEXT** if required information was not provided.
    Never deliver work you know is speculative without saying so.
```

**Controller notes:** Omit `model: "fast"` (or set the appropriate field) when you intentionally use a standard or strong implementer. Always paste the full task text and context into `prompt:`—do not substitute “read the plan” unless the task is explicitly scoped to discovery.
