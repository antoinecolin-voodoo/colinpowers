---
name: linear-integration
description: >-
  Use when working with Linear issue tracking alongside code: linking a session
  to an issue, searching the backlog, updating state and links, or posting
  targeted comments. Uses the Linear MCP server.
---

# Linear integration (MCP)

Coordinate agent work with **Linear** via the Linear MCP server. Tool calls below (`get_issue`, `save_issue`, `list_issues`, `save_comment`, `list_issue_statuses`) are the MCP tool names — invoke them with the platform-native MCP call:

- **Claude Code**: call the tool directly, e.g. `mcp__claude_ai_Linear__get_issue({ id: "CUP-123" })`.
- **Cursor**: wrap in `CallMcpTool { server: "user-Linear", toolName: "get_issue", arguments: { id: "CUP-123" } }`.

See `skills/using-workflow/references/cursor-tools.md` for the full mapping.

**Linked issue (session state):** Once an issue is chosen or confirmed, remember its identifier (e.g. `CUP-123`) for the rest of the task. If the user never links an issue, do not invent one.

---

## When starting work

Run this **after** the user’s goal is clear (often right after **brainstorming** picks a direction).

### Mode A — User gives a Linear issue ID

Examples: “Work on CUP-123”, “Fix LIN-456”.

1. **`get_issue`** — `{ "id": "<identifier>" }`  
   Optional: `includeRelations: true` if blocking/related context helps planning.
2. **`save_issue`** — `{ "id": "<identifier>", "state": "In Progress" }`  
   If the update fails or the name is unknown, call **`list_issue_statuses`** with `{ "team": "<team name or ID>" }` (from the issue or user), pick the closest equivalent to “In Progress”, and retry **`save_issue`** with that `state` value.
3. Use the issue **title** and **description** as primary context for planning and implementation.

**Do not** call **`save_issue`** without `id` (the only issues this skill creates are plan sub-issues; see “Sub-issues”).

### Mode B — No issue ID

1. Derive a short **search query** from the user’s task (key terms, feature names, error messages).
2. **`list_issues`** — e.g. `{ "query": "<keywords>", "limit": 25 }`  
   Add `team`, `state`, `assignee`, `project`, etc. only if the user or repo conventions imply them (e.g. filter to backlog / unstarted states when that matches how the team works).
3. **If results exist:** Present the **top 2–3** matches (identifier, title, one-line why it matches). Ask the user to **pick one** or **skip**.
   - **Pick:** treat as **Mode A** (fetch, set In Progress, proceed).
   - **Skip or no good match:** proceed **without** Linear tracking; do not keep searching unless the user asks.

---

## During work

| Situation | Action |
|-----------|--------|
| **Systematic debugging** (root-cause investigation) | If a Linear issue is linked: **`save_comment`** — `{ "issueId": "<id>", "body": "<concise markdown: hypothesis, evidence, conclusion / next step>" }`. One focused comment when the investigation has a useful outcome; not a live log. |
| **Design / spec clarification** with **user-facing impact** not reflected on the issue | If linked: **`save_comment`** with a **brief** note (what changed, why it matters). Otherwise skip. |
| Routine progress (“still working”, step-by-step updates) | **Do not** comment. |

---

## Sub-issues (big plans only)

When **subagent-development** runs a plan's workstreams and an issue is linked, each workstream becomes a sub-issue of it.

1. **Show the list** to the user: per workstream, title, one-line scope, and what it depends on. Create nothing before their go-ahead.
2. **Create them in dependency order** so blockers exist first. **`save_issue`** without `id`: `team` = the parent's team, `parentId` = the linked issue, `title`, `assignee` = the parent's assignee, `description` (scope, acceptance checks, plan path), `blockedBy` = the sub-issues it depends on.
3. Leave each one in its default state until its agent starts, then set it **In Progress** (and its handoff state when its branch merges, per **When finishing work**).

**No linked issue:** ask the user whether to link one or run the workstreams without Linear tracking.

---

## When finishing work

Run when implementation is done and the **git-branch-workflow** (or equivalent) would normally wrap up.

**If a Linear issue was linked for this session:**

1. **`save_issue`** — set the handoff state for the issue's team:
   - **Bug team** (workflow is Todo → In Progress → Fixed → Verified, no review state): `{ "id": "<identifier>", "state": "Fixed" }`. This is the explicit developer-handoff state in that team's flow; QA moves it to Verified from there.
   - **All other teams:** `{ "id": "<identifier>", "state": "In Review" }`.
   - If the state name is unknown or rejected, call **`list_issue_statuses`** with the issue's team, pick the closest developer-handoff equivalent, and retry. Do **not** fall through to a completed/verified state (e.g. "Verified", "Done", "Closed").
2. If a **PR URL** exists: **`save_issue`** — `{ "id": "<identifier>", "links": [{ "url": "<pr url>", "title": "<short title e.g. PR #123>" }] }`  
   `links` is append-only; include the PR once.

**If no issue was linked:** do nothing in Linear.

**Never** set state to Done/Closed/Verified or otherwise mark the fix as accepted; that stays a human decision. The **Bug** team's "Fixed" state is the one exception — it is the dev-side handoff in that team's workflow, not the accepted-by-QA state.

---

## Out of scope (do not do)

- Create **new** Linear issues, other than the approved sub-issues above.
- Change **estimate**, **assignee**, **delegate**, priority, cycles, or labels unless the user explicitly asks outside this skill.
- **Close** or **cancel** issues.
- Spam **progress** comments or duplicate PR links.

---

## Integration points

| Other skill / phase | How this skill ties in |
|---------------------|-------------------------|
| **Brainstorming** | After intent is clear, run **starting work** (Mode A or B) so planning uses issue context when available. |
| **Systematic debugging** | Post **`save_comment`** on the linked issue when investigation yields a durable finding. |
| **Subagent development** | Create one sub-issue per plan workstream (see **Sub-issues**); start and hand off each as its agent runs. |
| **Git branch workflow** (finish) | Run **When finishing work** so state and PR link stay in sync with the branch/PR. |

---

## MCP quick reference

| Agent action | `toolName` | Arguments (conceptual) |
|--------------|------------|-------------------------|
| Load issue | `get_issue` | `id` (required); optional `includeRelations`, `includeCustomerNeeds` |
| Update issue (state, links, …) | `save_issue` | `id` (required for updates); `state` optional; `links` optional `[{ url, title }]` |
| Create sub-issue | `save_issue` | no `id`; `team`, `parentId`, `title`, `description`, `assignee`, `blockedBy` (approved workstreams only) |
| Search / list issues | `list_issues` | `query`, `team`, `state`, `limit`, `assignee`, `project`, … |
| Add comment | `save_comment` | `issueId`, `body` (Markdown); `id` only when editing an existing comment |
| Resolve status names | `list_issue_statuses` | `team` (required) |

---

## Practical notes

- Prefer **exact issue identifiers** (`CUP-123`) in tool arguments.
- Keep comments **short**, **Markdown**, and **actionable** for humans scanning Linear later.
- If MCP calls fail (auth, permissions, network), tell the user and continue the coding task without blocking on Linear.
