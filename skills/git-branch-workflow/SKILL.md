---
name: git-branch-workflow
description: Use when starting or finishing feature work in a long-lived repo clone—creating a sub-feature branch off a parent feature branch, pushing it, squashing and cleaning up specs, updating Linear, and merging or opening a PR back to the parent (never main).
---

# Git branch workflow

Use this skill for **branch lifecycle** in projects where you stay in one clone (e.g. Unity): work happens on **sub-feature branches** branched from a **parent feature branch**, not from creating new clones per task.

---

## Starting work

1. **Detect the parent branch**  
   Run `git branch --show-current`. Treat this as the **parent branch** (e.g. `feature/guild-shop`).

2. **Confirm with the user**  
   If the current branch is ambiguous (`main`, `develop`, a stale name, or unclear intent), stop and ask which branch is the parent for this work.

3. **Derive a short branch slug**  
   From the task description or Linear issue **title** only. Use lowercase words, hyphens, no spaces.  
   **Do not** put Linear IDs (e.g. `CUP-123`) in the branch name—they do not describe the work.

4. **Create the sub-feature branch**  
   Pattern: `feature/<parent-segment>-<short-description>` with a **single hyphen chain** after `feature/`.  
   **Why:** Git cannot create `feature/guild-shop/cart` when `feature/guild-shop` already exists as a branch. Collapse the parent to a slug and join with hyphens, e.g. `feature/guild-shop-cart-fix`.

   Example parent `feature/guild-shop` + task "fix cart totals" -> `feature/guild-shop-cart-fix`.

5. **Create and push**  
   ```bash
   git checkout -b <sub-feature-branch>
   git push -u origin <sub-feature-branch>
   ```

6. **Remember the parent**  
   Store the parent branch name (conversation memory or notes) for the **Finishing** phase.

---

## Finishing work

### 0. Unity validation gate (mandatory)

**Goal before finishing:** Unity has refreshed the asset database (including `.meta` files where applicable) and the project compiles cleanly—or you have explicit human confirmation when automation cannot cover that.

**Do not start spec cleanup, squash, merge, or PR until this gate is satisfied** (via Coplay evidence and/or human confirmation, as below).

#### A. Automated path — Coplay MCP

When the **Coplay** MCP server is available, call its **`check_compile_errors`** tool (platform-native MCP syntax — see `skills/using-workflow/references/cursor-tools.md`). In practice this ties into the Unity Editor: it reloads the scripting domain, surfaces asset import side effects such as `.meta` generation/updates, and reports compilation errors.

1. Run **`check_compile_errors`** and read the full response.
2. If it reports compile errors, fix them (or report blocked state with evidence). **Do not** continue to finishing until the check is clean.
3. If the MCP call fails (server missing, Unity not connected, project root not set, auth, timeout, etc.), **do not** treat that as proof—fall through to **section B**.

#### B. Human validation — always when Coplay cannot replace it

**Stop and ask the user** (full manual gate) when **any** of these apply:

- Coplay is unavailable or **`check_compile_errors`** did not complete successfully.
- The work included **visual or scene-facing changes** that still need a human eye: UI layout, prefabs, scenes, materials, lighting, animation timing in context, or other editor-only judgment Coplay does not certify.
- Play Mode, runtime behavior, or acceptance criteria require the user to confirm in the Editor.

Use this prompt:

> All plan tasks are complete. Before I move to the finishing phase, please:
>
> 1. **Open the Unity Editor** on this project so it generates/updates `.meta` files (if not already done).
> 2. **Check the Console** for compilation errors and, if relevant, **verify visuals / Play Mode** for this change.
>
> Once you confirm everything is clean (or tell me what to fix), I'll commit `.meta` updates if any and proceed with squash, sync, and merge/PR.

Wait for the user's response. Do not proceed, guess, or skip this step when section B applies.

#### C. After the gate passes

- Run **`git status`**. Stage and commit new or changed **`.meta`** files (and any Unity-generated assets that belong in VCS): e.g. `git add '*.meta'` plus other paths as needed, then commit with an appropriate message (e.g. `chore: Unity meta and import updates`).
- Then continue to step 1 (Spec cleanup).

### 1. Spec cleanup

- Delete any `docs/specs/*.md` files that were introduced for this task.
- Stage the deletions: `git add -u docs/specs/` (or path-specific `git rm` as needed).

### 2. Squash commits (mandatory)

Goal: a **clean parent history**. Sub-feature work should land as one commit per Linear issue (or one descriptive commit if no issue).

| Situation | Action |
|-----------|--------|
| **One Linear issue** | Squash all commits into **one**. Message: `CUP-123: short description` (ID in message is fine; not in branch name). |
| **Multiple Linear issues** | Interactive rebase to **group commits per issue**. Propose the grouping to the user and get confirmation before rewriting history. |
| **No Linear issue** | Squash all into **one** commit with a clear imperative or conventional description. |

Use `git rebase -i` (onto parent or appropriate base) or squash merge strategy consistent with your local policy; the requirement is **one logical commit per issue** (or single commit) before merge/PR.

### 3. Sync with parent branch

Before merging or opening a PR, bring the parent's latest changes into the sub-feature branch:

1. `git fetch origin <parent-branch>`
2. `git merge origin/<parent-branch>` (into the current sub-feature branch)

**If merge conflicts occur:**

- **Simple conflicts** (a few files, clear resolution): describe the conflicts to the user, propose a resolution for each file, and apply only after the user confirms.
- **Complex conflicts** (many files, ambiguous intent, or risky resolutions): alert the user, list the conflicting files, and **stop**. Let the user resolve manually. Resume the finishing workflow once the user confirms conflicts are resolved and committed.

Do not force-resolve conflicts or guess intent. When in doubt, ask.

### 4. Linear update

- Set linked issue(s) to the dev-handoff state using the **`linear-integration`** skill. That skill picks the right state per team ("In Review" for most teams; "Fixed" for the Voodoo **Bug** team, which has no review state).

### 5. Merge (default) or PR

**Default: merge into parent (Option 1).** Proceed with the merge without asking.

**Ask the user to choose** between Option 1 and Option 2 only when **either**:

- The user has already said they want a PR for this work (honor that), **or**
- The parent branch is `develop`, `main`, `master`, or another shared long-lived integration branch. Merging directly into those usually skips team review; confirm the user really wants a direct merge rather than a PR.

Otherwise (the common case — merging a sub-feature branch back into another feature branch), run Option 1 directly.

**Option 1 -- Merge into parent (local integration) -- default**

1. `git checkout <parent-branch>`
2. `git pull` (fast-forward parent if applicable)
3. `git merge <sub-feature-branch>` (prefer fast-forward after squash; otherwise explicit merge)
4. `git push origin <parent-branch>` if the team pushes parent to remote
5. Delete sub-feature locally: `git branch -d <sub-feature-branch>`
6. Delete sub-feature on remote: `git push origin --delete <sub-feature-branch>`
7. **Stay on** `<parent-branch>`

No extra confirmation for branch deletion after merge -- this is the default for Option 1.

**Option 2 -- Open pull request**

1. Ensure squashed branch is pushed: `git push -u origin <sub-feature-branch>`
2. Create PR with GitHub CLI, **base = parent branch**, not `main`:

   ```bash
   gh pr create --base <parent-branch> --title "..." --body "..."
   ```

3. Summarize changes in the PR body (what changed, how to test, linked issues).
4. **Keep** the sub-feature branch on remote until the PR is merged; remote branch is removed by the hosting provider when the PR merges, if configured.

### 6. Checkout parent when done

After Option 1, you are already on parent. After Option 2, run `git checkout <parent-branch>` so the working tree matches ongoing feature work.

---

## Conventions (non-negotiable)

| Rule | Detail |
|------|--------|
| Branch names | Descriptive slugs from **titles** or task text -- **never** Linear IDs alone or as the primary name. |
| Squash | **Required** before merging to parent or opening PR. |
| Parent sync | **Required** after squash, before merge or PR. Resolve conflicts with user before proceeding. |
| PR base | **Parent feature branch** only -- not `main` / `master` unless the user explicitly overrides for an exception. |
| Branch deletion | Automatic after Option 1 merge (local + remote). Not deleted on Option 2 until merge via hosting UI. |

---

## Quick reference

| Phase | Key commands / actions |
|-------|-------------------------|
| Start | `git branch --show-current` -> confirm parent -> `git checkout -b feature/<parent-slug>-<desc>` -> `git push -u origin ...` |
| Specs | Remove task-local `docs/specs/*.md`, stage deletions |
| Squash | One issue -> one commit `CUP-123: ...`; many issues -> rebase grouping + user confirm; no issue -> one descriptive commit |
| Sync | `git fetch origin <parent>` -> `git merge origin/<parent>` into sub-feature; resolve conflicts with user |
| Linear | Dev-handoff state via `linear-integration` (In Review for most teams; Fixed for the Voodoo Bug team) |
| Merge (default) | checkout parent -> pull -> merge sub-feature -> push parent -> `branch -d` / `push origin --delete`. Skip the PR prompt unless parent is `develop`/`main`/`master` or the user asked for a PR. |
| PR (when asked or parent is shared integration branch) | `gh pr create --base <parent>` |

---

## Common mistakes

- **Nested branch names** like `feature/foo/bar` when `feature/foo` is already a branch -- Git rejects this. Use `feature/foo-bar-baz` instead.
- **Linear IDs in branch names** -- hard to read in logs and tabs; keep IDs in commit messages and PR titles where useful.
- **Opening PRs against `main`** while the real integration target is the long-lived feature branch -- breaks team flow and review scope.
- **Skipping squash** -- clutters parent with WIP and fixup commits.
- **Skipping parent sync** -- merging or opening a PR without pulling the latest parent risks hidden conflicts that surface later or break CI.
- **Forgetting `git push -u`** on a new branch -- next session or CI cannot see the branch.
- **Deleting the sub-feature branch before push** when the user chose PR -- lose the remote reference for the PR.

---

## Red flags

- Current branch is `main` (or production default) and the user did not confirm switching to a feature parent -- **do not** invent a sub-feature off `main` without explicit instruction.
- Skipping the **Unity validation gate** — never start spec cleanup, squash, or merge/PR without a clean **`check_compile_errors`** result (when Coplay is used successfully) **and** human confirmation when section B requires it; **and** committing applicable `.meta` / import artifacts after the gate.
- Uncommitted or untracked spec files outside `docs/specs/` -- do not delete blindly; only remove specs clearly tied to this task.
- **Force-push** on a branch others use -- coordinate before rewriting shared history.
- Multiple issues in one branch without a clear grouping -- get user confirmation before rebase/squash plan.
- No `gh` CLI and user wants PR -- fall back to web UI instructions with explicit base branch.
- **Merge conflicts during parent sync** -- never auto-resolve; always involve the user.

---

## Integration with other skills

| Caller | When |
|--------|------|
| **brainstorming** | After design is agreed, hand off **Starting work** steps: parent detection, branch name, `checkout -b`, push. |
| **subagent-development** / **executing-plans** | When a task slice is **done**, run **Finishing work**: spec cleanup, squash policy, parent sync, Linear handoff state, merge (default) or PR when parent is shared or the user asked, checkout parent. |

This skill **replaces** older flows that assumed a fresh clone per task or finishing only against `main`: one long-lived clone, parent feature branch, sub-feature branches, squash, merge or PR **to parent**.
