---
name: using-workflow
description: Use when starting any conversation — colinpowers entry point for skill routing, model tiers, rationalization checks, and Cursor tool discipline.
---

# Using Workflow

**Invoke relevant skills before any response or action.** If there is even roughly a 1% chance a skill applies to the situation, read that skill first — then answer, plan, or call tools.

## Skill routing

| Skill | When |
|-------|------|
| **using-workflow** | This entry skill — routing, models, red flags. |
| **brainstorming** | Creative or exploratory feature / behavior work before implementation. |
| **writing-plans** | You have a spec or multi-step requirements — before coding. |
| **executing-plans** | Running a written plan in order with checkpoints. |
| **subagent-development** | Same, but one **Task** subagent per separable task. |
| **dispatching-parallel** | Two or more independent tasks safe to run in parallel. |
| **git-branch-workflow** | **Before any implementation** (code changes, plan execution, bug fixes) — check current branch and create a sub-feature branch if needed. Also when integrating or closing branch work. |
| **linear-integration** | Linear issues / workflow via **call_mcp_tool** (MCP). |
| **requesting-code-review** | After meaningful work or before merge — structured review pass. |
| **receiving-code-review** | Acting on review comments before editing further. |
| **systematic-debugging** | Bugs, failures, or unexpected behavior — before speculative fixes. |
| **verification** | Before claiming done, fixed, or passing — run checks and cite fresh output. |
| **writing-skills** | Creating or editing colinpowers skills. |

## Implementation gate

Before writing or modifying **any** code (implementation, bug fix, refactor, plan execution), check your branch:

1. Run `git branch --show-current`.
2. If you are on `main`, `master`, or `develop` — **stop**. Invoke **git-branch-workflow** (Starting work) to create a sub-feature branch before touching code.
3. If you are on a feature branch and the task warrants a sub-feature branch (new issue, separable slice of work) — invoke **git-branch-workflow** (Starting work).
4. If you are already on the correct working branch — proceed.

This gate applies even when brainstorming was skipped (direct tasks, quick fixes, one-off requests). The only exception is when the user explicitly says to work on the current branch.

## Model selection

| Tier | Model | Use for |
|------|--------|--------|
| **fast** | composer-2-fast | Mechanical edits, 1–2 files, search, small refactors. |
| **standard** | claude-4.6-sonnet-medium | Multi-file work, integration, review, most debugging. |
| **strong** | claude-4.6-opus-high | Architecture, design tradeoffs, hard debugging, escalation. |

## Rationalization prevention

| Excuse | Reality |
|--------|--------|
| "I'll reply first, then open the skill." | Open and follow the skill first. |
| "This message doesn't need a skill." | If it might apply, read it. |
| "Too small to verify." | "Done" still needs evidence when you claim status. |
| "Should work / probably fine." | Run the relevant command; show output. |
| "Linter passed." | Linter ≠ compiler ≠ runtime. |
| "I'm confident." | Confidence is not verification. |
| "Just this once." | No exceptions. |
| "Different wording, same loophole." | Follow intent, not letter-pushing. |

## Tools

Prefer **Read**, **Write**, **StrReplace**, **Grep**, **Glob**, **SemanticSearch**, **Task**, **Shell**, **call_mcp_tool**, and other Cursor-native tools; choose the smallest set that fits the task.
