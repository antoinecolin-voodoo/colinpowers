---
name: linear-integration
description: >-
  Use when working with Linear issue tracking alongside code: linking a session
  to an issue, searching the backlog, updating state and links, or posting
  targeted comments. Applies MCP server `user-Linear`.
---

# Linear integration (MCP)

Coordinate agent work with **Linear** via the **`user-Linear`** MCP server. Invoke tools with **`CallMcpTool`**: set **`server`** to `user-Linear` and **`toolName`** to the tool below; pass arguments as JSON matching each tool’s schema.

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

**Do not** call **`save_issue`** without `id` (never create issues via this skill; see “Out of scope”).

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

## When finishing work

Run when implementation is done and the **git-branch-workflow** (or equivalent) would normally wrap up.

**If a Linear issue was linked for this session:**

1. **`save_issue`** — `{ "id": "<identifier>", "state": "In Review" }`  
   Resolve state name via **`list_issue_statuses`** if needed, same as for In Progress.
2. If a **PR URL** exists: **`save_issue`** — `{ "id": "<identifier>", "links": [{ "url": "<pr url>", "title": "<short title e.g. PR #123>" }] }`  
   `links` is append-only; include the PR once.

**If no issue was linked:** do nothing in Linear.

**Never** set state to Done/Closed or otherwise close the issue; that stays a human decision.

---

## Out of scope (do not do)

- Create **new** Linear issues (no `save_issue` without `id` for creation).
- Change **estimate**, **assignee**, **delegate**, priority, cycles, or labels unless the user explicitly asks outside this skill.
- **Close** or **cancel** issues.
- Spam **progress** comments or duplicate PR links.

---

## Integration points

| Other skill / phase | How this skill ties in |
|---------------------|-------------------------|
| **Brainstorming** | After intent is clear, run **starting work** (Mode A or B) so planning uses issue context when available. |
| **Systematic debugging** | Post **`save_comment`** on the linked issue when investigation yields a durable finding. |
| **Git branch workflow** (finish) | Run **When finishing work** so state and PR link stay in sync with the branch/PR. |

---

## MCP quick reference

| Agent action | `toolName` | Arguments (conceptual) |
|--------------|------------|-------------------------|
| Load issue | `get_issue` | `id` (required); optional `includeRelations`, `includeCustomerNeeds` |
| Update issue (state, links, …) | `save_issue` | `id` (required for updates); `state` optional; `links` optional `[{ url, title }]`; **never** create (no `id`-less create) |
| Search / list issues | `list_issues` | `query`, `team`, `state`, `limit`, `assignee`, `project`, … |
| Add comment | `save_comment` | `issueId`, `body` (Markdown); `id` only when editing an existing comment |
| Resolve status names | `list_issue_statuses` | `team` (required) |

---

## Practical notes

- Prefer **exact issue identifiers** (`CUP-123`) in tool arguments.
- Keep comments **short**, **Markdown**, and **actionable** for humans scanning Linear later.
- If MCP calls fail (auth, permissions, network), tell the user and continue the coding task without blocking on Linear.
