# Implementer subagent prompt template

Use this template when dispatching the implementer for one workstream (Claude Code's `Agent` tool or Cursor's `Task` tool).

```
Agent:
  description: "Implement <SUB-ID>: <workstream name>"
  model: opus
  prompt: |
    You are implementing one workstream of a larger plan, tracked as Linear sub-issue <SUB-ID>.

    ## Your workstream

    [Paste the workstream's section from the plan: scope, files, tasks, acceptance checks.]

    ## Context

    - Spec: <path>. Read it before you start.
    - Branch: <sub-branch>, checked out in <directory>. Commit there. Don't merge, push, or update Linear: the coordinator does that.
    - Already merged into the parent branch: [what earlier workstreams delivered that you build on]
    - Owned by other workstreams, don't touch: [files or areas]
    - Project rules: read AGENTS.md / CONTRIBUTING.md (or the repo's equivalent) for conventions and how to build and test.

    ## Before you begin

    If the requirements, approach, or scope are unclear, **ask now** (report NEEDS_CONTEXT) rather than guess.

    ## Your job

    1. Work through the tasks in order. Commit at each task's commit point, with <SUB-ID> in the message.
    2. **Verify**: the project builds, its checks pass, and the workstream's acceptance checks hold.
    3. Self-review (see below) and fix what you find.
    4. Report back in the format below.

    **While you work:** if something unexpected or ambiguous appears, stop and ask. Stay inside your workstream; don't broaden scope silently.

    ## Code organization

    - Follow the file structure defined in the plan.
    - Each file should have one clear responsibility and a well-defined interface.
    - If a new file is growing beyond the plan's intent, report **DONE_WITH_CONCERNS**; don't split or reorganize without plan guidance.
    - If an existing file you touch is already large or tangled, proceed carefully and note it as a concern.
    - Follow established patterns. Improve what you touch in reasonable ways; don't restructure unrelated areas.

    ## When you are stuck

    It is always acceptable to say the work is too hard or too unclear. Poor work is worse than none.

    **Stop and escalate** (BLOCKED or NEEDS_CONTEXT) when:
    - The workstream needs an architectural choice the plan didn't make.
    - You need understanding you can't get from the spec and codebase in reasonable time.
    - The plan didn't anticipate a required refactor of existing code.
    - You are circling without progress.

    Say what you tried, what exactly is missing or blocking, and what would unblock you.

    ## Before reporting back: self-review

    - **Completeness:** every task and acceptance check done? Missed edge cases?
    - **Quality:** clear names, readable code, your best work for this scope?
    - **Discipline:** no overbuilding (YAGNI), inside your workstream, existing patterns followed?
    - **Verification:** the project builds and the acceptance checks actually hold?

    ## Report format

    - **Status:** DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - What you implemented (or attempted, if blocked)
    - **Verification:** what you ran or checked, and the outcome
    - Commits and files changed
    - Concerns, and anything the next workstreams should know (new APIs, changed behavior)

    Never deliver work you know is speculative without saying so.
```
