---
name: using-workflow
description: Use when starting any conversation — colinpowers entry point for skill routing, model tiers, rationalization checks, and tool discipline. Works in Claude Code and Cursor.
---

# Using Workflow

**Invoke relevant skills before any response or action.** If there is even roughly a 1% chance a skill applies to the situation, read that skill first — then answer, plan, or call tools.

## Platform adaptation

Skills use **Claude Code tool names** as the canonical vocabulary (`Agent`, `Bash`, `Edit`, `Read`, `Write`, `Grep`, `Glob`, `TodoWrite`, direct `mcp__…` MCP calls).

- **Claude Code**: use those tools directly.
- **Cursor**: translate via `references/cursor-tools.md` in this skill folder (`Agent` → `Task`, `Bash` → `Shell`, `Edit` → `StrReplace`, direct MCP → `CallMcpTool { server, toolName, arguments }`, etc.).

When a skill mentions a tool name you don't have, check the mapping file — don't invent a tool and don't fail silently.

## Skill routing

| Skill | When |
|-------|------|
| **using-workflow** | This entry skill — routing, models, red flags. |
| **brainstorming** | Creative or exploratory feature / behavior work before implementation. |
| **writing-plans-lean** | You have a spec or multi-step requirements — before coding. Reference-first plans. |
| **executing-plans** | Running a written plan in order with checkpoints. |
| **subagent-development** | Same, but one subagent per separable task. |
| **dispatching-parallel** | Two or more independent tasks safe to run in parallel. |
| **git-branch-workflow** | **Before any implementation** (code changes, plan execution, bug fixes) — check current branch and create a sub-feature branch if needed. Also when integrating or closing branch work. |
| **linear-integration** | Linear issues / workflow via the Linear MCP server. |
| **requesting-code-review** | After meaningful work or before merge — structured review pass. |
| **receiving-code-review** | Acting on review comments before editing further. |
| **systematic-debugging** | Bugs, failures, or unexpected behavior — before speculative fixes. |
| **verification** | Before claiming done, fixed, or passing — run checks and cite fresh output. |
| **qa-product-changelog** | Produce a non-technical changelog of a branch for QA / product, with Linear links. |
| **writing-skills** | Creating or editing colinpowers skills. |

## Implementation gate

Before writing or modifying **any** code (implementation, bug fix, refactor, plan execution), check your branch:

1. Run `git branch --show-current`.
2. If you are on `main`, `master`, or `develop` — **stop**. Invoke **git-branch-workflow** (Starting work) to create a sub-feature branch before touching code.
3. If you are on a feature branch and the task warrants a sub-feature branch (new issue, separable slice of work) — invoke **git-branch-workflow** (Starting work).
4. If you are already on the correct working branch — proceed.

This gate applies even when brainstorming was skipped (direct tasks, quick fixes, one-off requests). The only exception is when the user explicitly says to work on the current branch.

## Model selection

Three capability tiers. Pick the model your platform exposes at that tier.

| Tier | Use for | Claude Code | Cursor |
|------|---------|-------------|--------|
| **fast** | Mechanical edits, 1–2 files, search, small refactors. | `claude-haiku-4-5` | `composer-2-fast` |
| **standard** | Multi-file work, integration, review, most debugging. | `claude-sonnet-4-6` | `claude-4.6-sonnet-medium` |
| **strong** | Architecture, design tradeoffs, hard debugging, escalation. | `claude-opus-4-7` | `claude-4.6-opus-high` |

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

Pick the smallest set that fits the task. Prefer native file, search, shell, and subagent tools over ad-hoc workarounds. When a skill names a specific tool, use the platform equivalent from `references/cursor-tools.md` if your platform names it differently.
