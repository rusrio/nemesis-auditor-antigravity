---
name: nemesis
description: "Runs a deep iterative security audit by alternating between the Feynman logic auditor and the State Inconsistency auditor until no new findings emerge. Use this when the user asks for a security audit, says 'nemesis audit', '/nemesis', 'deep audit', or 'audit this codebase'. Language-agnostic: works on Solidity, Move, Rust, Go, C++, Python, TypeScript."
---

# N E M E S I S
### The Inescapable Auditor

```
    ╔═══════════════════════════════════════════════════════════════╗
    ║                                                               ║
    ║   "Nemesis — the goddess of divine retribution against        ║
    ║    those who succumb to hubris."                              ║
    ║                                                               ║
    ║   Your code was written with confidence.                      ║
    ║   Nemesis questions that confidence.                          ║
    ║   Then maps what your confidence forgot to protect.           ║
    ║   Then questions it again.                                    ║
    ║                                                               ║
    ║   Nothing survives both passes.                               ║
    ║                                                               ║
    ╚═══════════════════════════════════════════════════════════════╝
```

## Goal

Orchestrate a full iterative security audit loop. Alternate between two methodologies — Feynman (logic bugs) and State Inconsistency (coupled state desync) — feeding each pass's findings into the next, until the loop converges.

---

## Instructions

### Step 0 — Parse flags

Before starting, check the user's message for optional flags:
- `--contract [Name]` → scope the entire audit to that single contract/module only
- `--pass1` → run ONLY the Feynman methodology (skip State)
- `--pass2` → run ONLY the State methodology (requires `outputs/feynman-pass1.md` to exist)
- `--continue` → resume from the last completed pass found in `outputs/`

### Step 1 — Recon (Phase 0)

Before reading code in detail, do an attacker mindset scan of the codebase:

```
Q0.1 ATTACK GOALS: What is the worst an attacker can achieve?
     List the top 3–5 catastrophic outcomes.

Q0.2 NOVEL CODE: What is NOT a fork of battle-tested code?
     Custom math, novel state machines, unique mechanisms = highest bug density.

Q0.3 VALUE STORES: Where does value actually sit?
     Map every module that holds funds, assets, or accounting state.
     For each: what code path moves value OUT? What authorizes it?

Q0.4 COMPLEX PATHS: What is the most complex interaction path?
     Paths crossing 4+ modules with 3+ external calls = prime targets.

Q0.5 COUPLED VALUE: Which value stores have dependent accounting?
     For each store from Q0.3: what other storage must stay in sync?
     Build initial coupling hypothesis before reading code deeply.
```

Write the recon output to: `outputs/recon.md`

---

### Step 2 — Pass 1: Feynman Audit (full run)

Load and execute the full Feynman methodology from `.agent/skills/feynman/SKILL.md`.

Input: codebase + `outputs/recon.md`

Save output to: `outputs/feynman-pass1.md`

Extract from the output:
- Verified findings
- SUSPECT state variables and functions
- Exposed assumptions (implicit trusts about state)
- Ordering concerns (external call timing issues)

---

### Step 3 — Pass 2: State Inconsistency Audit (full run, enriched)

Load and execute the full State Inconsistency methodology from `.agent/skills/state/SKILL.md`.

Input: codebase + `outputs/feynman-pass1.md`

Enrichment from Pass 1:
- Feynman SUSPECTS → add as extra state audit targets
- Exposed assumptions → reveal new coupled pairs to map
- Ordering concerns → check if a state gap exists at the flagged point
- Function-State Matrix → use as base for the Mutation Matrix

Save output to: `outputs/state-pass2.md`

Extract from the output:
- State GAPS (functions missing coupled updates)
- New coupled pairs discovered via Feynman enrichment
- Masking code flagged (ternary clamps, min caps)
- Parallel path mismatches

---

### Step 4 — Convergence loop (targeted passes)

After Pass 2, check: did the last pass produce ANY new findings, coupled pairs, suspects, or root causes not in previous passes?

**IF YES → continue:**

**Pass 3 (Feynman targeted):** Load `.agent/skills/feynman/SKILL.md`. Scope ONLY to functions/state touched by Pass 2's new output. Do NOT re-audit what Pass 1 already cleared. Save to `outputs/feynman-pass3.md`.

**Pass 4 (State targeted):** Load `.agent/skills/state/SKILL.md`. Scope ONLY to new coupled pairs and mutation paths from Pass 3. Do NOT re-audit what Pass 2 already cleared. Save to `outputs/state-pass4.md`.

Repeat alternating passes until no new findings emerge.

**IF NO → converged.** Proceed to Step 5.

**Safety limit:** Maximum 6 total passes (3 Feynman + 3 State).

---

### Step 5 — Final consolidation

1. Merge all pass outputs into a unified finding set
2. Deduplicate findings with the same root cause
3. Verify all CRITICAL/HIGH/MEDIUM findings (code trace + PoC trigger sequence)
4. Tag each finding with its discovery path:
   - `Feynman-only` — found in Pass 1, never enriched by State
   - `State-only` — found in Pass 2, never enriched by Feynman
   - `Cross-feed P[N]→P[M]` — found via back-and-forth interaction
5. Save final report to: `outputs/nemesis-verified.md`

---

### Step 6 — Final report format

Write `outputs/nemesis-verified.md` using this structure:

```markdown
# N E M E S I S — Verified Findings

## Scope
- Language: [detected]
- Modules analyzed: [list]
- Coupled state pairs mapped: [N]
- Mutation paths traced: [N]
- Nemesis loop iterations: [N]

## Verification Summary
| ID | Source | Severity | Verdict |
|----|--------|----------|---------|
| NM-001 | Feynman→State | HIGH | TRUE POSITIVE |

## Verified Findings

### Finding NM-001: [Title]
**Severity:** CRITICAL | HIGH | MEDIUM | LOW
**Source:** [Feynman-only | State-only | Cross-feed Pass N→Pass M]
**Verification:** [Code trace | PoC sequence | Both]

**Root Cause:** [What is broken and why]
**Trigger Sequence:**
1. [Step-by-step minimal exploit]

**Impact:** [Fund loss, locked state, incorrect accounting, etc.]
**Fix:** [Minimal code change]

---

## Feedback Loop Discoveries
[Findings that ONLY emerged from cross-feed between auditors]

## False Positives Eliminated
[Findings that failed verification, with explanation]
```

---

## Constraints

- NEVER invent code that doesn't exist in the codebase
- NEVER assume a coupled pair without finding code that reads BOTH values together
- NEVER claim a function is missing an update without tracing its full call chain
- NEVER report a finding without a trigger sequence AND downstream consequence
- NEVER skip the convergence loop — cross-feed passes produce the highest-value bugs
- ALWAYS present only verified findings in the final report
- ALWAYS show exact file paths and line numbers for every finding
