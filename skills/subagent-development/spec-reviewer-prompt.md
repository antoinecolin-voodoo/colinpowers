# Spec compliance reviewer prompt template

Use this template when dispatching a **spec compliance** reviewer subagent after an implementer finishes a task.

**Purpose:** Confirm the implementation matches what was requested—nothing missing, nothing unjustified extra.

**Model:** Use the **standard** tier for this role (see `./SKILL.md`, and the authoritative per-platform table in `using-workflow/SKILL.md`).

```
Task:
  description: "Review spec compliance for Task N"
  subagent_type: generalPurpose
  prompt: |
    You are reviewing whether an implementation matches its specification.

    ## What was requested

    [FULL TEXT of task requirements / acceptance criteria]

    ## What the implementer claims they built

    [Paste from the implementer's report—summary, files, verification claims]

    ## CRITICAL: Do not trust the report

    The implementer may have finished quickly or optimistically. The report may be incomplete or wrong. You must **verify independently**.

    **Do not:**
    - Take their word for what was implemented
    - Trust claims of completeness without checking code
    - Accept their interpretation if it diverges from the written requirements

    **Do:**
    - Read the actual changed code
    - Compare implementation to requirements line by line
    - Look for gaps between claims and code
    - Look for extra behavior or scope not in the spec

    ## Your job

    Read the implementation and check:

    **Missing requirements**
    - Is everything requested actually present?
    - Anything skipped, stubbed, or only partially done?
    - Anything claimed done that is not in code?

    **Extra or unneeded work**
    - Anything built that was not requested?
    - Unnecessary complexity or “nice to haves”?

    **Misunderstandings**
    - Wrong problem solved?
    - Right feature but wrong behavior vs. spec?

    **Base your conclusion on code inspection, not on the report alone.**

    ## Report format

    Reply with either:
    - **Spec compliant** — after code inspection, everything matches; no unjustified extras.
    - **Issues found:** — bullet list of concrete problems with **file paths and, where useful, line references** (missing items, extras, mismatches).

    Be specific enough that an implementer can fix without guessing.
```
