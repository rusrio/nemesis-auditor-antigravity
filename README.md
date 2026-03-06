# N E M E S I S

### The Inescapable Auditor

```
    +===============================================================+
    |                                                               |
    |   "Nemesis -- the goddess of divine retribution against       |
    |    those who succumb to hubris."                              |
    |                                                               |
    |   Your code was written with confidence.                      |
    |   Nemesis questions that confidence.                          |
    |   Then maps what your confidence forgot to protect.           |
    |   Then questions it again.                                    |
    |                                                               |
    |   Nothing survives both passes.                               |
    |                                                               |
    +===============================================================+
```

An iterative deep-logic security audit agent for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Nemesis combines two complementary audit methodologies in an alternating feedback loop to find bugs that neither can catch alone. **Language-agnostic** -- works on Solidity, Move, Rust, Go, C++, Python, TypeScript, or any codebase.

---

## How It Works

Nemesis is an **agent** -- an orchestrator that runs an iterative back-and-forth loop between two sub-agents:

| Pass | Agent | What It Finds |
|------|-------|---------------|
| 1 | **Feynman Auditor** | Business logic bugs via first-principles questioning. Every line challenged. Every assumption exposed. |
| 2 | **State Inconsistency Auditor** | Coupled state desync bugs. Maps every mutation path. Finds gaps where one side updates without the other. |
| 3+ | **Alternating targeted passes** | Each pass feeds findings into the next. Feynman suspects become state audit targets. State gaps become Feynman interrogation points. |

The loop runs until **convergence** -- no new findings in a pass (max 6 passes).

### Why Both?

- **Feynman alone** finds logic bugs but may miss structural state gaps
- **State Inconsistency alone** finds desync bugs but may miss WHY state was designed that way
- **Nemesis** runs them back and forth -- each pass feeds the next -- finding bugs at every iteration that the previous pass missed

---

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed and configured
- A codebase you want to audit

---

## Installation

### Option A: Copy into your project (recommended)

```bash
# Clone this repo
git clone https://github.com/sainikethan/nemesis-auditor.git

# Copy the .claude folder into your project
cp -r nemesis-auditor/.claude /path/to/your-project/

# You can delete the cloned repo after copying
rm -rf nemesis-auditor
```

### Option B: Run from this repo

```bash
# Clone and enter the repo
git clone https://github.com/sainikethan/nemesis-auditor.git
cd nemesis-auditor

# Symlink your codebase into this directory
ln -s /path/to/your/contracts ./contracts
```

---

## Usage

### 1. Start Claude Code in your project directory

```bash
cd /path/to/your-project   # must contain the .claude/ folder
claude
```

### 2. Launch the audit

Type any of these in the Claude Code prompt:

```
/nemesis                        # Full iterative audit (recommended)
/nemesis --contract MyToken     # Audit a single contract
/nemesis --pass1                # Only run the Feynman Auditor
/nemesis --pass2                # Only run the State Inconsistency Auditor
```

### 3. What happens next

Once you type `/nemesis`, the agent will:

1. **Phase 0 -- Recon:** Scan the codebase, identify entry points, build an attacker's hit list
2. **Pass 1 -- Feynman Auditor:** Read every function line-by-line, asking "WHY does this line exist? What breaks if it changes? What assumption does this rely on?" Produces findings, suspects, and exposed assumptions
3. **Pass 2 -- State Inconsistency Auditor:** Map every coupled state pair (balance <-> checkpoint, stake <-> rewardDebt, etc.), trace every mutation path, and find every gap where one side updates without the other. Uses Pass 1's suspects as extra targets
4. **Pass 3+ -- Iterative refinement:** Feynman re-interrogates Pass 2's gaps. State re-checks Pass 3's new findings. Alternates until nothing new surfaces
5. **Final -- Consolidation:** Deduplicates, verifies all Critical/High/Medium findings, and produces the final report

### 4. Read the results

Findings are saved to `.audit/findings/`:

```
.audit/findings/
  feynman-pass1.md          # Pass 1 Feynman findings
  state-pass2.md            # Pass 2 State Inconsistency findings
  feynman-pass3.md          # Pass 3 targeted Feynman (if needed)
  state-pass4.md            # Pass 4 targeted State (if needed)
  nemesis-verified.md       # Final consolidated + verified report
```

### 5. Resume if interrupted

If the audit gets interrupted mid-way:

```
/nemesis --continue             # Picks up from the last completed pass
```

---

## All Commands

| Command | What It Does |
|---------|-------------|
| `/nemesis` | Full iterative audit until convergence |
| `/nemesis --pass1` | Pass 1 only -- full Feynman Auditor |
| `/nemesis --pass2` | Pass 2 only -- State Inconsistency on existing Pass 1 output |
| `/nemesis --continue` | Resume from where the last pass left off |
| `/nemesis --contract [name]` | Full audit scoped to a single contract |
| `/feynman` | Run the Feynman Auditor standalone (no iteration) |
| `/state-audit` | Run the State Inconsistency Auditor standalone (no iteration) |

---

## Finding Format

Each finding in the report includes:

```markdown
### Finding NEM-001: [Title]
**Severity:** CRITICAL | HIGH | MEDIUM | LOW
**Discovery Path:** Feynman-only | State-only | Cross-feed Pass N -> Pass M

**Root Cause:** [What is broken and why]
**Trigger Sequence:**
1. [Step-by-step to reproduce]

**Impact:** [What goes wrong -- fund loss, locked state, etc.]
**Fix:** [Minimal code change]
**Verification:** Code trace | PoC test | Both
```

---

## Architecture

Nemesis is built from 3 Claude Code skills:

```
.claude/
  skills/
    nemesis-auditor/
      SKILL.md              # The orchestrator -- runs the iterative loop
    feynman-auditor/
      SKILL.md              # First-principles logic bug finder
    state-inconsistency-auditor/
      SKILL.md              # Coupled state desync detector
```

### Feynman Auditor

Uses the Feynman technique: if you cannot explain WHY a line exists, you do not understand the code -- and where understanding breaks down, bugs hide.

- **Phase 0:** Attacker mindset recon (what's worth stealing? what's the kill chain?)
- **Phase 1:** Scope and inventory -- build Function-State Matrix
- **Phase 2:** Per-function interrogation (7 categories, 28+ questions per function)
- **Phase 3:** Cross-function analysis (guard consistency, inverse parity)
- **Phase 4:** Synthesize raw findings
- **Phase 5:** Verification gate (eliminate false positives)

### State Inconsistency Auditor

Systematically finds bugs where an operation mutates one piece of coupled state without updating its dependent counterpart.

- **Phase 1:** Map all coupled state pairs (balance <-> checkpoint, stake <-> rewardDebt, etc.)
- **Phase 2:** Find every mutation path for each state variable
- **Phase 3:** Cross-check -- does each mutation update ALL coupled state?
- **Phase 4:** Check operation ordering within functions
- **Phase 5:** Compare parallel code paths (withdraw vs liquidate, transfer vs burn)
- **Phase 6:** Trace multi-step user journeys for stale state accumulation
- **Phase 7:** Detect masking/defensive code that hides broken invariants
- **Phase 8:** Verification gate (eliminate false positives)

---

## Language Support

Nemesis is **language-agnostic**. Logic bugs live in the reasoning, not the syntax.

| Language | Status |
|----------|--------|
| Solidity | Fully supported |
| Move (Sui/Aptos) | Fully supported |
| Rust | Fully supported |
| Go | Fully supported |
| C++ | Fully supported |
| Python | Fully supported |
| TypeScript | Fully supported |

---

## License

MIT
