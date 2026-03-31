---
name: systematic-debugging
description: >-
  Use when encountering any bug, test failure, or unexpected behavior, before
  proposing fixes. Four-phase root-cause process; no fixes without investigation.
---

# Systematic debugging

## Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Core principle:** Always find root cause before attempting fixes. Symptom fixes are failure.

**Violating the letter of this process is violating the spirit of debugging.**

## The iron law

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

If you have not completed Phase 1, you cannot propose fixes.

## When to use

Use for **any** technical issue:

- Test failures
- Bugs in production
- Unexpected behavior
- Performance problems
- Build failures
- Integration issues

**Use this especially when:**

- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You have already tried multiple fixes
- Previous fix did not work
- You do not fully understand the issue

**Do not skip when:**

- Issue seems simple (simple bugs have root causes too)
- You are in a hurry (rushing guarantees rework)
- Someone wants it fixed now (systematic is faster than thrashing)

## Model guidance

- **Start** debugging at the **standard** model (`claude-4.6-sonnet-medium`).
- **Escalate** to the **strong** model (`claude-4.6-opus-high`) if:
  - Phase 2 does not surface a clear pattern (working vs broken comparison stays unclear), or
  - Phase 3 hypotheses keep failing after disciplined minimal tests.

Hard or cross-cutting bugs often need stronger reasoning; do not burn time guessing at the wrong tier.

## The four phases

You **must** complete each phase before proceeding to the next.

### Phase 1: Root cause investigation

**Before attempting any fix:**

1. **Read error messages carefully**
   - Do not skip past errors or warnings
   - They often contain the exact solution
   - Read stack traces completely
   - Note line numbers, file paths, error codes

2. **Reproduce consistently**
   - Can you trigger it reliably?
   - What are the exact steps?
   - Does it happen every time?
   - If not reproducible -> gather more data, do not guess

3. **Check recent changes**
   - What changed that could cause this?
   - Git diff, recent commits
   - New dependencies, config changes
   - Environmental differences

4. **Gather evidence in multi-component systems**

   **When the system has multiple components (CI -> build -> signing, API -> service -> database):**

   **Before proposing fixes, add diagnostic instrumentation:**

   ```
   For EACH component boundary:
     - Log what data enters the component
     - Log what data exits the component
     - Verify environment/config propagation
     - Check state at each layer

   Run once to gather evidence showing WHERE it breaks
   THEN analyze evidence to identify the failing component
   THEN investigate that specific component
   ```

   **Example (multi-layer system):**

   ```bash
   # Layer 1: Workflow
   echo "=== Secrets available in workflow: ==="
   echo "IDENTITY: ${IDENTITY:+SET}${IDENTITY:-UNSET}"

   # Layer 2: Build script
   echo "=== Env vars in build script: ==="
   env | grep IDENTITY || echo "IDENTITY not in environment"

   # Layer 3: Signing script
   echo "=== Keychain state: ==="
   security list-keychains
   security find-identity -v

   # Layer 4: Actual signing
   codesign --sign "$IDENTITY" --verbose=4 "$APP"
   ```

   **This reveals:** Which layer fails (e.g. secrets -> workflow OK, workflow -> build FAIL).

5. **Trace data flow**

   **When the error is deep in the call stack:** bugs often show up far from where they start (wrong path, wrong config, uninitialized value). **Do not fix only where the exception appears.**

   **Backward tracing (inline):**

   - Observe the symptom (message, bad value, failed assertion).
   - Identify the immediate line or function that failed.
   - Ask: what called this, and with what arguments?
   - Walk up the chain until you find the **original trigger** (first bad assumption, missing setup, wrong input).
   - **Fix at the source**, not at the deepest symptom.

   If manual tracing is hard, add **temporary instrumentation** immediately *before* the risky operation: log inputs, environment, and `new Error().stack` (in tests, `console.error` often surfaces better than a quiet logger). Use the output to see which caller or test produced the bad state.

   **Defense in depth (after you know the root cause):** invalid data can re-enter via another path. After fixing the source, consider **validation at each layer** the data crosses: entry/API boundaries, business logic, environment guards (e.g. "refuse dangerous ops in test CI"), and targeted debug logs for the next incident. One check is easy to bypass; several aligned checks make recurrence unlikely.

   **Condition-based waiting (flaky timing):** if the issue is "sometimes fails," replace arbitrary `sleep`/fixed delays with **waiting until the condition you actually need** is true (poll with a bounded timeout and a clear timeout message). Only use fixed delays when timing *is* what you are testing -- and document why.

### Phase 2: Pattern analysis

**Find the pattern before fixing:**

1. **Find working examples**
   - Locate similar working code in the same codebase
   - What works that is similar to what is broken?

2. **Compare against references**
   - If implementing a pattern, read the reference implementation **completely**
   - Do not skim -- read every line
   - Understand the pattern fully before applying

3. **Identify differences**
   - What is different between working and broken?
   - List every difference, however small
   - Do not assume "that cannot matter"

4. **Understand dependencies**
   - What other components does this need?
   - What settings, config, environment?
   - What assumptions does it make?

### Phase 3: Hypothesis and testing

**Scientific method:**

1. **Form a single hypothesis**
   - State clearly: "I think X is the root cause because Y"
   - Write it down
   - Be specific, not vague

2. **Test minimally**
   - Make the **smallest** possible change to test the hypothesis
   - One variable at a time
   - Do not fix multiple things at once

3. **Verify before continuing**
   - Did it work? Yes -> Phase 4
   - Did not work? Form a **new** hypothesis
   - **Do not** stack more fixes on top

4. **When you do not know**
   - Say "I do not understand X"
   - Do not pretend to know
   - Ask for help
   - Research more

### Phase 4: Implementation

**Fix the root cause, not the symptom:**

1. **Reproduce the bug reliably**
   - Confirm you can trigger it **consistently** before applying any fix
   - Same steps, same environment, same observable failure
   - If reproduction is intermittent, narrow conditions (flags, timing, data) until it is dependable enough to verify a fix

2. **Decide: implement directly or delegate**

   The main agent (you) ran Phases 1-3 and holds all investigation context. Now choose who implements the fix:

   | Situation | Action |
   |-----------|--------|
   | **Simple fix** (1-2 files, clear change, light context) | Implement directly -- delegation overhead is not worth it. |
   | **Heavier fix** (multiple files, or your context is already long from investigation) | **Delegate** to a subagent via **Task**. This keeps your context clean for verification and potential re-investigation. |

   **When delegating:** Craft a focused subagent prompt containing:
   - The **root cause** (one sentence: what is broken and why)
   - The **confirmed hypothesis** from Phase 3
   - **Exact files** to modify and what the change should do
   - **Reproduction steps** so the subagent can verify its own fix
   - **Constraints**: one change at a time, no "while I am here" improvements, no bundled refactoring

   Use the **fast** model for mechanical fixes (1-2 files, straightforward edit) and the **standard** model for multi-file or judgment-heavy fixes. Do **not** dump your full investigation history into the subagent -- curate context, not copy context.

   **When implementing directly:**
   - Address the root cause you identified
   - **One** change at a time
   - No "while I am here" improvements
   - No bundled refactoring

3. **Verify the fix**
   - **You** (the main agent) always verify -- never trust a subagent's self-report alone
   - Does the failure stop under the same reproduction?
   - No regressions in adjacent behavior?
   - Issue actually resolved in the scenario that mattered?
   - If a subagent implemented the fix, review its diff before running verification

4. **If the fix does not work**
   - **Stop**
   - Count: how many fixes have you tried?
   - If fewer than 3: return to Phase 1, re-analyze with new information
   - **If 3 or more: stop and question the architecture (step 5 below)**
   - **Do not** attempt fix #4 without architectural discussion
   - If a subagent implemented a failed fix, do **not** re-delegate blindly -- re-investigate first, then delegate again with updated context if needed

5. **If 3+ fixes failed: question architecture**

   **Patterns suggesting an architectural problem:**

   - Each fix reveals new shared state, coupling, or breakage elsewhere
   - Fixes require large refactors to "make it fit"
   - Each fix creates new symptoms in a different place

   **Stop and question fundamentals:**

   - Is this pattern fundamentally sound?
   - Are we persisting through inertia?
   - Should we refactor architecture instead of continuing symptom fixes?

   **Escalate** for architectural discussion with your team (and consider the **strong** model) before more local fixes.

   This is not a failed hypothesis -- it may be the wrong shape of solution.

### After Phase 4: Linear issue comment

**If a Linear issue is linked** to this work, add a **comment** on that issue summarizing the root cause investigation:

- What you found (evidence, failing layer, trace)
- What actually caused the bug
- How it was fixed (and how you verified)

Use the **`linear-integration`** skill and the Linear MCP tools to post that comment so the issue record stays useful for the team.

## Red flags -- stop and follow process

If you catch yourself thinking:

- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "Add multiple changes, run tests"
- "Skip repro, I will eyeball it"
- "It is probably X, let me fix that"
- "I do not fully understand but this might work"
- "Pattern says X but I will adapt it differently"
- "Here are the main problems:" [lists fixes without investigation]
- Proposing solutions before tracing data flow
- **"One more fix attempt" (when you already tried 2+)**
- **Each fix reveals a new problem in a different place**

**All of these mean: stop. Return to Phase 1.**

**If 3+ fixes failed:** question the architecture (Phase 4, step 5).

## Common rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, do not need process" | Simple issues have root causes too. The process is fast for simple bugs. |
| "Emergency, no time for process" | Systematic debugging is **faster** than guess-and-check thrashing. |
| "Just try this first, then investigate" | First fix sets the habit. Do it right from the start. |
| "I will confirm repro after the fix lands" | If you cannot trigger it before the fix, you cannot prove the fix worked. |
| "Multiple fixes at once saves time" | You cannot isolate what worked. It causes new bugs. |
| "Reference too long, I will adapt the pattern" | Partial understanding guarantees bugs. Read it completely. |
| "I see the problem, let me fix it" | Seeing symptoms != understanding root cause. |
| "One more fix attempt" (after 2+ failures) | 3+ failures -> architectural problem. Question the pattern, do not fix again blindly. |
| "I'll just implement it myself, faster than delegating" | If your context is heavy and the fix is multi-file, delegation keeps you sharp for verification and re-investigation. |

## Quick reference

| Phase | Key activities | Success criteria |
|-------|----------------|------------------|
| **1. Root cause** | Read errors, reproduce, check changes, gather evidence, trace data flow | Understand **what** and **why** |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Reliable repro, delegate or implement directly, verify | Bug resolved; verification evidence |

## When process reveals "no root cause"

If systematic investigation shows the issue is truly environmental, timing-dependent, or external:

1. You have still completed the process
2. Document what you investigated
3. Implement appropriate handling (retry, timeout, clearer error surfacing)
4. Add monitoring/logging for future investigation

**But:** most "no root cause" claims are incomplete investigation.

## Real-world impact

From debugging practice:

- Systematic approach: often resolves in a focused block of time
- Random fixes: long thrash, rework, and new defects
- First-time fix rate and regression risk improve sharply when fixes follow evidence

## Related skills

- **`verification`** -- Before claiming fixed or passing: run the relevant checks and cite fresh output.
- **`linear-integration`** -- When a Linear issue is linked: update state, links, and **post the root-cause summary comment** after the fix.
- **`subagent-development`** -- Phase 4 delegation uses the same **Task** tool; for multi-file fixes, craft a focused subagent prompt rather than implementing in a cluttered context.
