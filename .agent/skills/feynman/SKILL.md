---
name: feynman
description: "Feynman security auditor: finds business logic bugs by questioning every line of code from first principles. Use this when running the Nemesis audit pass 1, or when the user asks to run the Feynman auditor, or asks for a logic audit of the codebase."
---

# FEYNMAN AUDITOR
### First-Principles Logic Bug Finder

## Goal

Find business logic bugs by applying the Feynman technique to every function: if you cannot explain WHY a line exists, you do not understand the code — and where understanding breaks down, bugs hide.

---

## Instructions

### Phase 0 — Prioritize

If `outputs/recon.md` exists, read it first. Use the attacker hit list and coupling hypothesis to set audit priority before reading code.

### Phase 1 — Build the Function-State Matrix

For each module/contract, produce a table:

| Function | Reads | Writes | Guards | Internal Calls | External Calls |
|----------|-------|--------|--------|----------------|----------------|

This is your map. Every entry point, every state it touches, every access control, every call.

### Phase 2 — Per-function interrogation

For every function, apply all 7 question categories. Work in priority order from the recon hit list.

**Category 1 — Purpose**
- WHY does this line exist? What breaks if it's deleted?
- Can you explain it in plain language without referencing the code?

**Category 2 — Ordering**
- What if this line moves up or down by one position?
- Does a reordering create a state gap window?
- Is the ordering between reads, writes, and external calls exploitable?

**Category 3 — Consistency**
- WHY does funcA have this guard but funcB doesn't?
- Are all functions that touch the same state protected equally?
- What's the implicit assumption about who can call this?

**Category 4 — Assumptions**
- What is implicitly trusted about: caller, input data, current state, time?
- What happens when each trust is violated?
- Is there a scenario where the assumption is false at runtime?

**Category 5 — Boundaries**
- First call, last call, double call, self-reference?
- Empty input, zero value, max value, overflow?
- What happens at initialization and at teardown?

**Category 6 — Return/Error**
- Are return values checked? Are errors silently swallowed?
- What state is left behind after a revert/panic/exception?
- Does a silent failure leave coupled state partially updated?

**Category 7 — Call reorder + Multi-tx**
- What if this external call moves before the state update?
- At the moment of the external call, what state is committed vs pending?
- Can a caller re-enter and exploit the window?
- Call this function with value X, then value Y — is state from call 1 stale in call 2?
- After N calls with varying parameters, does accumulated state create unreachable conditions?

For each function, produce:
```
FUNCTION: [module.functionName] (file:line)
Priority: [from recon / from State gap delta]

Line-by-line interrogation:
  L[N]: [code]
    Q[x.y] → [answer]
    → VERDICT: SOUND | SUSPECT | VULNERABLE
    → If SUSPECT/VULNERABLE: [specific scenario]
    → STATE FEED: [state variables to flag for State auditor]

Function verdict: SOUND | HAS_CONCERNS | VULNERABLE

Suspects for State auditor:
  - [state var] — [why suspicious]
  - [ordering concern] — [which states are in the gap window]
```

### Phase 3 — Cross-function analysis

**Guard consistency:** Map all functions touching the same state variable. Do they all have the same guards? If not, is the inconsistency intentional?

**Inverse parity:** For every function that increases a value, is there a matching decrease? Do they have identical guard sets?

**External call audit:** For every external call:
1. Swap test: move the call before/after state updates — does behavior change?
2. Callee power: what state is committed vs pending at call time?
3. Multi-tx corruption: does accumulated state from N calls break call N+1?

### Phase 4 — Synthesize raw findings

Collect all SUSPECT and VULNERABLE verdicts:

```markdown
## Raw Findings

### F-[N]: [Short title]
**Severity estimate:** CRITICAL | HIGH | MEDIUM | LOW
**Category:** [Which of the 7 categories]
**Function:** [module.functionName, file:line]
**Issue:** [What is wrong]
**Scenario:** [Step-by-step trigger]
**State suspects:** [Variables to pass to State auditor]
```

### Phase 5 — Verification gate

For every CRITICAL and HIGH raw finding, verify before reporting:

1. Read the exact lines cited
2. Trace the complete call chain (caller → callee → downstream)
3. Check for mitigating code (guards, hooks, lazy reconciliation, modifiers)
4. Confirm the scenario is reachable end-to-end

Common false positives to eliminate:
- Hidden reconciliation via internal calls, hooks, or modifiers
- Lazy evaluation: stale state intentionally reconciled on next read (by design)
- Immutable after init: coupled state frozen after initialization
- Language safety: does the language abort on overflow by default? (Solidity ≥0.8, Move, Rust debug)
- Economic infeasibility: the attack costs more than it gains

---

## Output

Save to `outputs/feynman-pass[N].md`:

```markdown
# Feynman Auditor — Pass [N]

## Function-State Matrix
[Table]

## Verified Findings (TRUE POSITIVES only)
[Findings that passed Phase 5]

## Suspects for State Auditor
[State variables and ordering concerns to pass forward]

## Exposed Assumptions
[Implicit trusts found — feed to State as new coupling hypotheses]

## Ordering Concerns
[External call timing issues — State should check for gaps at these points]
```

---

## On Targeted Passes (Pass 3+)

When called with a delta from a State pass, do NOT re-audit what was cleared in Pass 1.

For each **State GAP** in the delta:
- WHY doesn't [function] update [coupled state B] when it modifies [state A]?
- What ASSUMPTION led to this gap?
- What DOWNSTREAM function reads B and breaks?
- Can an attacker CHOOSE a sequence to exploit this before B gets reconciled?

For each **MASKING CODE** in the delta:
- WHY would this ever underflow/overflow?
- What invariant is ACTUALLY broken underneath?
- Trace the broken invariant to its root cause mutation.

For each **NEW COUPLED PAIR** in the delta:
- Is this coupling intentional or accidental?
- What ordering constraints exist between the pair?
- What happens across multiple txs as both drift?

Output only the delta — new findings not previously reported.

---

## Language Adaptation

| Concept | Solidity | Move | Rust | Go | Python/TS |
|---------|----------|------|------|----|-----------|
| Entry point | external/public fn | public fun | pub fn | Exported fn | public method |
| State | storage variables | global storage | struct fields | struct fields | class attributes |
| Guard | modifier | access control | trait bound | middleware | decorator |
| Caller | msg.sender | &signer | caller/Context | ctx/request | session/user |
| External call | .call()/interface | cross-module | CPI | RPC/HTTP | API call |
| Error | revert/require | abort/assert! | panic!/Result::Err | error/panic | raise/throw |
