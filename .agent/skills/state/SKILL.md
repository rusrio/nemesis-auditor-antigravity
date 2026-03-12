---
name: state
description: "State Inconsistency Auditor: finds bugs where an operation mutates one piece of coupled state without updating its dependent counterpart. Use this when running the Nemesis audit pass 2, or when the user asks for a state audit, coupled state analysis, or desync bug detection."
---

# STATE INCONSISTENCY AUDITOR
### Coupled State Desync Detector

## Goal

Find bugs where an operation mutates one piece of coupled state without updating its dependent counterpart. Two state variables are "coupled" when they represent the same underlying fact from different angles — updating one without the other leaves the system holding contradictory truth.

### Common coupled pairs to look for:
- `balance ↔ checkpoint` — balance changes but checkpoint not updated → stale rewards
- `stake ↔ rewardDebt` — stake changes but rewardDebt not reset → reward inflation/deflation
- `totalShares ↔ userShares[addr]` — total changes but individual doesn't (or vice versa)
- `position.size ↔ position.healthFactor` — size changes but health not recomputed
- `reserves[0] ↔ reserves[1]` — one reserve updates before the other in a swap
- `accRewardPerShare ↔ userRewardDebt[addr]` — accumulator advances but user debt not snapshotted

---

## Instructions

### Phase 1 — Map all coupled state pairs

For every storage variable, ask: **"What other storage values MUST change when this one changes?"**

Build the **Coupled State Dependency Map**:

```
State A changes → State B MUST change (invariant: [relationship])
State C changes → State D AND State E MUST change
```

Look for:
- per-user balance ↔ per-user accumulator/tracker/checkpoint
- numerator ↔ denominator
- position size ↔ position-derived values (health, rewards, shares)
- total/aggregate ↔ sum of individual components
- any cached computation ↔ inputs it was derived from
- any global index/accumulator ↔ last-snapshot of that index per user
- mapping[key] ↔ array tracking the same keys

**Enrichment from Feynman:** For each Feynman exposed assumption like "dev assumes X stays in sync with Y", add X↔Y as a coupled pair hypothesis and verify.

### Phase 2 — Build Mutation Matrix

For each state variable, list every function that modifies it (direct writes, increments, decrements, deletions, indirect via internal calls, hooks, callbacks).

| State Variable | Mutating Function | Updates Coupled State? |
|----------------|-------------------|-----------------------|
| [var] | [function] | ✓ SYNCED / ✗ GAP |

**Enrichment from Feynman:** Add Feynman SUSPECT state variables as extra rows, even if you would not have identified them from state mapping alone.

### Phase 3 — Cross-check: does each mutation update ALL coupled state?

For each cell where a function writes State A:
1. Check if the function also writes State B (coupled counterpart)
2. If not: is there a modifier, hook, or internal call that does?
3. If still not: mark as GAP
4. For each GAP: is this lazy evaluation by design? Check docs/comments.
5. If not lazy: FINDING

### Phase 4 — Operation ordering within functions

Trace the exact order of state changes in each function:

```
step 1: reads A and B → computes result
step 2: modifies B based on result
step 3: [external call]        ← gap: B updated, A not yet
step 4: modifies A
```

At each step:
- Are ALL coupled pairs consistent RIGHT HERE?
- Does step N use a value that step N-1 already invalidated?
- If an external call happens between steps, can the callee observe inconsistent state?

**Enrichment from Feynman:** Check if a state gap exists at each ordering point Feynman flagged.

### Phase 5 — Parallel path comparison

Group functions that achieve similar outcomes (transfer vs burn, withdraw vs liquidate, normal vs emergency path, single vs batch). For each group: do ALL paths update the SAME coupled state?

| Coupled State | Path A | Path B | Path C |
|---------------|--------|--------|--------|
| [pair] | ✓/✗ | ✓/✗ | ✓/✗ |

### Phase 6 — Multi-step user journey tracing

Trace adversarial sequences (adapt to the actual codebase):

- Deposit → partial withdraw → claim rewards (rewards computed on which balance?)
- Stake → unstake half → restake → unstake all (reward debt correct through each step?)
- Open position → add collateral → partial close → health check (health updated at each step?)
- Provide liquidity → swaps happen → remove liquidity (fee tracking correct through reserve changes?)
- Borrow → partial repay → borrow again → check debt (accumulator rebased at each step?)

For each journey: write the state of every coupled pair after each step. Highlight any step where one side advances without the other.

### Phase 7 — Detect masking/defensive code

Look for code that HIDES a broken invariant rather than fixing it:
- ternary clamps: `value > 0 ? value : 0`
- min/max caps: `Math.min(computed, balance)`
- try/catch swallowing errors silently
- early returns that skip updates on edge cases

For each masking pattern: flag it and pass to the Feynman auditor for root cause interrogation. The mask signals a broken invariant underneath.

### Phase 8 — Verification gate

For every CRITICAL and HIGH gap, verify before reporting:

1. Trace the full call chain — is there a hidden update via modifier/hook/base class?
2. Is this lazy evaluation by design? Check NatSpec and comments.
3. Is the coupled state immutable after init?
4. Is the "gap" an intentional asymmetry documented in the protocol?

Common false positives:
- Hidden reconciliation via `_beforeTokenTransfer` hook, `_updateReward` modifier
- Lazy evaluation: state reconciled on next READ, not every WRITE (by design)
- Immutable after init: both sides frozen after initialization
- Designed asymmetry: states intentionally not coupled the way assumed

---

## Output

Save to `outputs/state-pass[N].md`:

```markdown
# State Inconsistency Auditor — Pass [N]

## Coupled State Dependency Map
[All coupled pairs with invariant description]

## Mutation Matrix
[Table: state var × mutating function × sync status]

## Parallel Path Comparison
[Table: coupled pair × similar functions × synced or gap]

## State GAPS Found
[Functions that update one side of a coupled pair but not the other]

## Masking Code Flagged
[Defensive patterns signaling broken invariants — pass to Feynman]

## New Coupled Pairs (from Feynman enrichment)
[Pairs not visible from state alone]

## Verified Findings (TRUE POSITIVES only)
[Gaps that passed Phase 8 verification]

## Delta for Next Feynman Pass
[New gaps, masking code, coupled pairs not in previous passes]
```

---

## On Targeted Passes (Pass 4+)

When called with a delta from a Feynman pass, do NOT re-audit what was cleared in Pass 2.

For each **NEW SUSPECT** from Feynman's delta:
- Is this state variable part of a coupled pair?
- Does the suspect function update all counterparts?
- Does Feynman's root cause analysis reveal additional gaps?

For each **ROOT CAUSE** from Feynman's delta:
- Trace the root cause mutation through ALL code paths
- Check parallel paths for the same root cause pattern
- Check if the root cause affects other coupled pairs not yet mapped

Output only the delta — new findings not previously reported.
