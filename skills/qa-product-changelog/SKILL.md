---
name: qa-product-changelog
description: Use when asked to produce a branch changelog or release notes for QA, product, or other non-technical stakeholders.
---

# QA / Product changelog from a branch

Produce a **stakeholder-facing** changelog from a git branch over a range, with each change linked to its Linear issue when one exists. Audience is **QA and product people, not engineers** — translate technical wording, and keep bug fixes crisp so testers know what to re-verify.

Uses the Linear MCP server for issue lookup (`get_issue`, `list_issues`). Call tools via whichever syntax your platform provides — see `skills/using-workflow/references/cursor-tools.md` for the Claude Code ↔ Cursor mapping. Assumes a git clone of the repo is checked out locally.

---

## 1. Ask for the range (always first)

Do **not** guess the range. Ask the user for a start and end boundary, accepting either form:

- **Commits** — SHAs or refs, e.g. `abc1234..HEAD`, `v1.2.0..feature/foo`.
- **Dates** — e.g. "since Thursday afternoon", "between 2026-04-16 and 2026-04-20". Convert relative phrasing to absolute dates (today's date is known from context) before running git.

Also confirm the **branch** if it is not obvious (the current branch is usually right; the user may point to a different one).

Prompt template:

> What range should I summarize? You can give me:
> - commit SHAs or refs (e.g. `abc1234..HEAD`), or
> - dates (e.g. "since 2026-04-16 afternoon").
>
> And which branch should I look at? (default: current branch)

---

## 2. Get the full commit log

**Gotcha:** the Bash tool wrapper can truncate `git log` output with bodies. Always use `--no-pager`, redirect to a temp file, and read via the Read tool. Writing to a file avoids truncation.

**Dates:**

```bash
git --no-pager log \
  --since="<YYYY-MM-DD HH:MM>" --until="<YYYY-MM-DD HH:MM>" \
  --pretty=format:"===%n%h%n%s%n%b%n" <branch> \
  > /tmp/commits_full.txt
```

**Commit range:**

```bash
git --no-pager log \
  --pretty=format:"===%n%h%n%s%n%b%n" <start>..<end> \
  > /tmp/commits_full.txt
```

**If output is still truncated:** some Claude Code configurations wrap `git` with a proxy that intercepts bare `git` invocations (e.g. token-optimization hooks). Bypass with `command git …`, `$(command -v git) …`, or the full path from `which git` — do not hard-code `/usr/bin/git`, which is Apple's Xcode stub on macOS.

Then Read `/tmp/commits_full.txt`. Verify the line count looks reasonable (compare against `--oneline | wc -l`) — if the file is suspiciously short, the output was truncated and you need to re-run via the direct binary.

---

## 3. Extract Linear issue IDs

Scan subjects **and** bodies for every reference form teams actually use:

- Plain IDs: `LIVO-527`, `BUG-3339`
- Bracketed: `[BUG-3193]`
- Linear URLs: `https://linear.app/<org>/issue/BUG-3339`
- Multiple per commit: `(LIVO-500, LIVO-522)` — count both

Collect the **unique set** of IDs across all commits, and remember which commit(s) reference each ID so you can describe the actual change later.

Note IDs that look like other trackers (e.g. `ART-1306`) — they are likely **not** in Linear; keep them as plain text in the output, not as a Linear link.

---

## 4. Fetch Linear details in parallel

Call the `get_issue` MCP tool (platform-native syntax — see the mapping file) once per unique ID, **in parallel** — not sequentially.

Keep: `id`, `title`, `url`, `status`, `description` (for wording cues), `project`.

If a referenced ID 404s, keep it in the output as plain text with the ID but no link.

---

## 5. Classify commits

Walk every commit and bucket it:

| Bucket | Examples | Action |
|--------|----------|--------|
| **Has Linear ID** | `LIVO-527`, `BUG-3339` | Use Linear title + translate description for non-tech audience |
| **Meaningful, no ID** | "Fix rewards containers position", "Add ally achievement" | Keep for ticketless section — try to match against backlog in step 6 |
| **Internal-only** | "Remove Debug logs", merge commits, "fix: set ci_ref to main", build/CI plumbing | Drop unless user asks for full detail |
| **Too vague** | "Improvement", "fix" with no body | Drop or ask user |

Rule of thumb for QA/product: if a tester or PM cannot verify or care about it, drop it.

### Drop already-verified bug issues

Exclude any Linear issue from the **`Bug`** team whose status is **`Verified`**. QA has already signed off, and listing them wastes re-verification time.

- Use the `team` and `status` fields returned by `get_issue` in step 4.
- If every commit associated with an issue only carried that already-verified ID (no other ticket, no substantive unticketed change), drop those commits entirely.
- If the commit also did meaningful work beyond the verified fix, keep it in the ticketless section with a plain description.
- This rule is team-specific: **only** the `Bug` team's `Verified` state. Do not filter out `Done` / `Completed` statuses from feature teams — those are in-scope for the report.

---

## 6. Match ticketless commits against the backlog

Commit authors routinely forget to cite the Linear issue. Before declaring a commit "no ticket", search for one.

### 6a. Pick a search scope

Ask the user, then combine filters as needed. Pick the **narrowest** scope that still covers the work — narrow scopes are cheap to scan, broad scopes flood the context and force `jq` workarounds.

| Scope | `list_issues` filter | Good when |
|-------|-----------------------|-----------|
| **Project** | `project: "<name>"` | Branch is a single feature with a dedicated Linear project (e.g. "Guild Hall"). |
| **Team** | `team: "<name>"` | Work spans projects but stays within one team (e.g. a Bug team triaging across features). |
| **Cycle / sprint** | `cycle: "<number or name>"` | You want only issues being worked this sprint. |
| **Assignee** | `assignee: "<email or name>"` or `"me"` | One person's work across projects. |
| **Recent activity** | `updatedAt: "-P14D"` (ISO-8601 duration) | Cross-cutting branch with no single project; bound by when the commits landed. |
| **Label** | `label: "<name>"` | Feature/epic is labeled rather than projected. |
| **Free-text per commit** | `query: "<keywords>"`, `limit: 10` | Last resort when no structural scope fits — search per commit subject, one call each. |

**Default strategy when the user doesn't know:**
1. Ask for **project first** (most feature branches have one).
2. If the branch touches multiple areas, add **recent activity** (`updatedAt`) on top of team or assignee.
3. For the few commits still unmatched after scoped search, do a **per-commit `query` search** with 2–3 keywords from the subject.

### 6b. Run the search

```
list_issues { "project": "<name>", "limit": 250 }          # or team / cycle / …
```

**Gotcha:** large results can exceed the tool-result size limit and get saved to a file. Extract just the fields you need with `jq`:

```bash
/usr/bin/jq -r '.[0].text | fromjson | .issues[] |
  "\(.id) | \(.status) | \(.title) | \(.url)"' <saved-file> > /tmp/backlog.txt
```

### 6c. Match carefully

For each ticketless commit, look for a backlog issue whose **title matches the commit's intent**, not just overlapping keywords. When unsure, call `get_issue` to read the description before accepting a match.

**Only match when confident.** A weak match is worse than no match — QA will chase the wrong ticket. Leave weak matches in the ticketless section.

---

## 7. Format the output

Two sections, in this order.

### Section A — Changes with Linear tickets

One bullet per issue. The **whole "ID: Title" string** is the markdown link to the issue URL, followed by an em dash and a plain-English description of what changed:

```
- [LIVO-527: Remove guild XP from the Rewards panel for Achievements](https://linear.app/voodoo/issue/LIVO-527) — XP is granted server-side, so only the coin reward is displayed on the claim popup
```

Writing the description:

- **Translate for non-engineers.** Drop class names, method names, and internal systems ("NakamaService", "RectMask2D", "cellIdentifier"). Replace with player-facing language.
- **Make bug fixes concrete** so QA knows exactly what to re-verify. Describe the **observable symptom that was fixed**, not the code change.
- If several commits share one issue, describe the combined outcome (one bullet per issue, not per commit).

### Section B — Changes without a Linear ticket

Split into two subsections:

- **Bug fixes** — with enough detail that QA can reproduce/verify.
- **Product / visual changes** — what a player would notice.

Plain bullets, no Linear link. If there is a non-Linear tracker reference (e.g. `ART-1306`), keep it in parentheses at the end.

```
- Fixed guild quest timer showing 00:00 (timing deltas were inverted)
- Added new visual elements on max-level guild hall, plus a new feature to change guild hall flags color
- Removed curved arches (ART-1306)
```

---

## Quick reference

| Phase | Action |
|-------|--------|
| Range | Ask for commits **or** dates + branch; convert relative dates to absolute. |
| Log | `/usr/bin/git --no-pager log … > /tmp/commits_full.txt` then Read. |
| IDs | Regex scan for `[A-Z]+-\d+` + Linear URLs in subject and body. |
| Fetch | `get_issue` per ID, **in parallel**. |
| Backlog cross-ref | Pick narrowest scope (project / team / cycle / assignee / recent / label / query) → `list_issues` → `jq` the saved file → match confidently only. |
| Classify | Has ID / meaningful no-ID / internal / vague. Drop internal + vague. **Drop `Bug` team issues already in `Verified`.** |
| Format | Section A (linked) + Section B (Bug fixes, Product/visual). Translate technical language. |

---

## Common mistakes

- **Guessing the range.** If the user said "since Thursday" without a date, ask or confirm the absolute date — do not pick one silently.
- **Bash truncation.** Piping `git log` with `%b` through the Bash tool can silently drop commits. Always write to a temp file via `/usr/bin/git --no-pager` and Read it.
- **One Linear issue = one bullet.** If three commits all cite `LIVO-500`, write **one** bullet describing the combined change, not three.
- **Technical descriptions leaking through.** "Added null check in NakamaService" is useless to QA. Rewrite as "fixed crash when claiming a completed guild quest".
- **Sequential Linear fetches.** Always fetch issues in parallel — one round trip per ID wastes time.
- **Weak backlog matches.** If the title is not a clear fit for the commit intent, leave it in the ticketless section. Do not force a match.
- **Listing merge and debug-cleanup commits.** QA does not care. Drop them.
- **Inventing Linear links.** If an ID references a tracker that is not Linear (e.g. `ART-…`) or the fetch 404s, write it as plain text, not as a link.

---

## Red flags

- Commit range you derived returns **zero commits** or suspiciously few — your date/commit boundaries are probably wrong. Confirm with the user before continuing.
- Commit message references an ID that does **not exist in Linear** — surface this to the user; may be a typo in the commit, or an ID from a different tracker.
- Linear `list_issues` returns a very large payload — use `jq` on the saved file instead of loading the full response into context.
- Ticketless commits that look like meaningful product changes but the user never mentioned a scope — ask which project / team / cycle / assignee / label (or just a date window) to search before concluding "no ticket". Free-text `query` per commit is a last-resort fallback.

---

## Integration with other skills

| Other skill | How this ties in |
|-------------|-------------------|
| **linear-integration** | This skill calls `get_issue` and `list_issues` on the same Linear MCP server. Reuse its conventions for tool calls. |
| **git-branch-workflow** | Typical trigger: after finishing a feature branch and before sharing progress with QA/product, generate this changelog for the range since the branch diverged or since the last share. |
