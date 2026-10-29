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

**Do not start spec cleanup, squash, merge, or PR until this gate is satisfied** (via Unity CLI evidence and/or human confirmation, as below).

The gate runs on Unity's own local tooling: the **Unity CLI** (`unity`) driving the **Unity Pipeline package** (`com.unity.pipeline`), which hosts a local HTTP API inside the running Editor. **No MCP server is involved** — do not drive Unity from the shell any other way, and never launch the Editor binary directly.

The command names below are the ones the pipeline registers (snake_case). Confirm what your project's Editor actually exposes with **`unity list`** (~140 commands), and read the project's own `AGENTS.md` / `CONTRIBUTING.md` first — project recipes win over this skill.

Docs: [Unity Pipeline package](https://docs.unity.com/en-us/unity-production-pipeline/local-tools-cli/unity-pipeline-package) · [Unity CLI reference](https://docs.unity.com/en-us/unity-cli/unity-cli-reference)

#### A. Prerequisites and CLI syntax

```bash
unity --version        # CLI present (Unity Hub installs it; otherwise: brew install --cask unity-cli)
unity auth status      # signed in -- run `unity auth login` if not
unity pipeline list    # expect "Pipeline: Installed" (com.unity.pipeline in Packages/manifest.json)
```

- Editor **6.0 or later** is required by the pipeline package. If the package is missing: open the project in the Editor, run `unity pipeline install`, wait for the recompile, re-run `unity pipeline list`.
- **Args are `--key value` *after* the command name.** `key=value` is silently ignored and the command then runs with defaults — e.g. `unity cmd menu path=…` lists every menu item instead of executing one.
- **Global flags go *before* `cmd`** (`unity --json cmd …`, `--no-banner`, `--quiet`); after it they are forwarded to the Editor command as parameters.
- With `--json`, some commands double-encode `result` as a JSON string — parse twice when needed.
- Long operations are async: trigger, then poll the matching `*_status` command. Destructive commands need `--confirm true` and usually support `--dry_run true`.
- Prereqs failing means the automated path is unavailable — go to **section D**, do not fake the gate.

#### B. A Unity Editor already has this worktree open — drive that instance

Check first:

```bash
unity status           # connected Editors: port, project, version, state
```

If a connected Editor has **this worktree** open, use it. **Never start a batch-mode run against a project an interactive Editor holds** — the two fight over the project lock.

Compile check:

```bash
unity cmd clear_console                                   # isolate this run's logs
unity cmd menu --path "Assets/Refresh"                    # import changed assets, write .meta, trigger recompile
unity cmd recompile_status                                # poll until completed / up_to_date
unity cmd get_console_logs --severity error --limit 50    # errors only; avoid bare `console` (full stack traces)
```

1. `unity cmd recompile` forces a recompile when the refresh does not pick the change up.
2. Poll **`recompile_status`** until `completed` or `up_to_date`; its **`failed`** flag and **`errors[]`** array are the gate's evidence.
3. Compile errors: fix them, or report the blocked state with the CLI output as evidence. Only a **fresh** clean status counts — output from an earlier session is not evidence either way.
4. During a domain reload the HTTP server drops. Retry until **`editor_status`** reports `ready`; a dropped connection is not a failed check. If the Editor stalls while unfocused, `set_autotick` keeps it ticking.
5. Tests on the live Editor when the change warrants them: `unity cmd run_tests --mode editor --filter <name> --timeout 240` (discover names with `list_tests`; long runs take `--async_tests true`, then poll `test_status`).
6. No connected Editor, or the CLI path fails outright → **section C**, then **section D**.

#### C. No Editor open — start one, or run the headless gate

- **Start an interactive Editor:** `unity open .` launches the correct Editor version and returns immediately; poll `unity status` until the Editor appears `ready` (cold boot with import can take minutes). Then run **section B**.
- **Headless gate:** `unity test . --mode EditMode --filter <name>` spawns a *separate* batch-mode Editor — the reliable choice for long PlayMode/determinism gates, and valid **only** when no interactive Editor holds the project.

Read the CLI **exit code**, not the log prose:

| Exit | Meaning for the gate |
|------|----------------------|
| `0` | Pass. |
| `6` | Run never finished — compile error, licence, Editor crash, or `--timeout`. **Not** a pass; investigate the output. |
| `8` | `unity test` only: tests ran and some failed. Fix them; retrying reproduces the same result. |
| `7` | Unity service unreachable. Retryable — retry once, then treat as no evidence and go to **section D**. |

#### D. Human validation — always when the CLI cannot replace it

**Stop and ask the user** (full manual gate) when **any** of these apply:

- The CLI path did not complete successfully: CLI missing, not signed in, pipeline package absent, no Editor that can be started or connected, or an exit code / `errors[]` you cannot resolve.
- The work included **visual or scene-facing changes** that still need a human eye: UI layout, prefabs, scenes, materials, lighting, animation timing in context, or other editor-only judgment the CLI does not certify. (`capture_game_view` / `capture_scene_view` screenshots inform that judgment; they do not replace it.)
- Play Mode, runtime behavior, or acceptance criteria require the user to confirm in the Editor.

Use this prompt:

> All plan tasks are complete. Before I move to the finishing phase, please:
>
> 1. **Open the Unity Editor** on this project so it generates/updates `.meta` files (if not already done).
> 2. **Check the Console** for compilation errors and, if relevant, **verify visuals / Play Mode** for this change.
>
> Once you confirm everything is clean (or tell me what to fix), I'll commit `.meta` updates if any and proceed with squash, sync, and merge/PR.

Wait for the user's response. Do not proceed, guess, or skip this step when section D applies.

#### E. After the gate passes

- Run **`git status`**. Stage and commit new or changed **`.meta`** files (and any Unity-generated assets that belong in VCS): e.g. `git add '*.meta'` plus other paths as needed, then commit with an appropriate message (e.g. `chore: Unity meta and import updates`). **Never hand-write a `.meta` file** — let the refresh above generate it.
- **Keep editor-side noise out of the commit.** Driving the Editor over the pipeline's HTTP server can dirty project settings unrelated to your change — e.g. the server sets `Application.runInBackground`, which writes `runInBackground: 1` into `ProjectSettings/ProjectSettings.asset`. Review the diff and `git checkout -- <file>` any such artifact before committing.
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
- The parent branch is `develop`, `main`, or `master`. Merging directly into those usually skips team review; confirm the user really wants a direct merge rather than a PR.

Release branches (e.g. `release/x.y.z`) are treated like any other parent feature branch: run Option 1 directly without asking.

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

0. **Required:** run the pre-PR review from **requesting-code-review** (Before opening a pull request). Squash its fixes into the branch's commit(s), and don't open the PR while a finding still needs the user's call.
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
- Skipping the **Unity validation gate** — never start spec cleanup, squash, or merge/PR without a clean Unity CLI result (`unity cmd menu --path "Assets/Refresh"` then `recompile_status` with no `errors[]`, or a headless `unity test .` at exit `0`) **and** human confirmation when section D requires it; **and** committing applicable `.meta` / import artifacts after the gate.
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
