---
name: verification
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs — requires running verification commands and confirming output before any success claims; evidence before assertions always
---

# Verification Before Completion

## Overview

Claiming work is complete without verification is dishonesty, not efficiency.

**Core principle:** Evidence before claims, always.

**Violating the letter of this rule is violating the spirit of this rule.**

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Compiles | Build output: 0 errors | Previous run, "should compile" |
| Unity compiles + import refresh | Fresh **`check_compile_errors`** (Coplay MCP `user-coplay-mcp`) when available and successful; otherwise human confirmation in Editor | Assuming compile without tool output or user sign-off |
| Works in editor | Manual Play Mode verification steps | Code looks right, no run |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Reproduce original symptom: resolved | Code changed, assumed fixed |
| Acceptance criteria met | Line-by-line checklist verification | Single happy path only |
| Requirements met | Line-by-line checklist | Build passing alone |
| Agent completed | VCS diff shows changes | Agent reports "success" |

## Red Flags — STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!", etc.)
- About to commit/push/PR without verification
- Trusting agent success reports
- Relying on partial verification
- Thinking "just this once"
- Tired and wanting work over
- **ANY wording implying success without having run verification**

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion ≠ excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |

## Key Patterns

**Build:**

```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**Unity (Coplay):**

```
✅ [call_mcp_tool user-coplay-mcp / check_compile_errors] [See: clean compile in response] + commit .meta if changed
❌ Skip human check when visuals/scenes/UI changed and Coplay cannot validate appearance or Play Mode
```

**Acceptance criteria:**

```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Build passes, task complete" without criteria check
```

**Agent delegation:**

```
✅ Agent reports success → Check diff → Verify changes → Report actual state
❌ Trust agent report alone
```

## Why This Matters

- Trust erodes when claims don't match reality
- Undefined or broken code can ship and crash at runtime
- Incomplete features ship when completion is assumed
- False completion wastes time on redirect and rework
- Honesty about state is a core working agreement

## When To Apply

**ALWAYS before:**

- ANY variation of success/completion claims
- ANY expression of satisfaction
- ANY positive statement about work state
- Committing, PR creation, task completion
- Moving to next task
- Delegating to agents

**Rule applies to:**

- Exact phrases
- Paraphrases and synonyms
- Implications of success
- ANY communication suggesting completion/correctness

## The Bottom Line

**No shortcuts for verification.**

Run the command. Read the output. THEN claim the result.

This is non-negotiable.
