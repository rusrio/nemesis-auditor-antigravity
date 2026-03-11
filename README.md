# N E M E S I S

### The Inescapable Auditor — Antigravity Edition

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

An iterative deep-logic security audit agent system for [Antigravity IDE](https://antigravity.dev). Nemesis combines two complementary audit methodologies in an alternating feedback loop to find bugs that neither can catch alone. **Language-agnostic** — works on Solidity, Move, Rust, Go, C++, Python, TypeScript, or any codebase.

> This is a fork of [nemesis-auditor](https://github.com/0xiehnnkta/nemesis-auditor) adapted for Antigravity.
> The original Claude Code version (`.claude/` folder) is preserved for cross-compatibility.

---

## How It Works

Nemesis runs an **iterative back-and-forth loop** between three agents:

| Agent | Role | What It Finds |
|-------|------|---------------|
| `nemesis` | Orchestrator | Runs the loop, manages passes, writes final report |
| `feynman` | Sub-agent | Business logic bugs via first-principles questioning |
| `state` | Sub-agent | Coupled state desync bugs — maps every mutation path |

The loop alternates Feynman → State → Feynman → State until **convergence** (no new findings), max 6 passes.

---

## Requirements

- [Antigravity IDE](https://antigravity.dev) installed and configured
- A codebase to audit

---

## Installation

### Copy into your project (recommended)

```bash
# Clone this repo
git clone https://github.com/rusrio/nemesis-auditor-antigravity.git

# Copy the .antigravity folder into your project
cp -r nemesis-auditor-antigravity/.antigravity /path/to/your-project/

# Create the outputs directory
mkdir -p /path/to/your-project/outputs

# You can delete the cloned repo after copying
rm -rf nemesis-auditor-antigravity
```

### Or run from this repo

```bash
git clone https://github.com/rusrio/nemesis-auditor-antigravity.git
cd nemesis-auditor-antigravity

# Symlink your codebase
ln -s /path/to/your/contracts ./contracts

# Open in Antigravity
antigravity .
```

---

## Usage

### 1. Open your project in Antigravity

```bash
cd /path/to/your-project   # must contain the .antigravity/ folder
antigravity .
```

### 2. Launch the audit

Type any of these in the Antigravity chat:

```
nemesis audit                           # Full iterative audit (recommended)
nemesis audit --contract MyToken        # Audit a single contract
nemesis audit --pass1                   # Only run the Feynman agent
nemesis audit --pass2                   # Only run the State agent
nemesis audit --continue                # Resume if interrupted
```

### 3. What happens next

Once you trigger the audit, the `nemesis` agent will:

1. **Phase 0 — Recon:** Attacker mindset scan — builds hit list and initial coupling hypothesis → saves to `outputs/recon.md`
2. **Pass 1 — Feynman Agent:** Reads every function line-by-line, questioning WHY each line exists. Produces findings, suspects, and exposed assumptions → saves to `outputs/feynman-pass1.md`
3. **Pass 2 — State Agent:** Maps every coupled state pair, traces every mutation path, finds gaps where one side updates without the other. Uses Pass 1's suspects as extra targets → saves to `outputs/state-pass2.md`
4. **Pass 3+ — Iterative refinement:** Feynman re-interrogates State's gaps. State re-checks Feynman's new findings. Alternates until nothing new surfaces.
5. **Final — Consolidation:** Deduplicates, verifies all Critical/High/Medium findings, produces final report → saves to `outputs/nemesis-verified.md`

### 4. Read the results

Findings are saved to `outputs/` in your project:

```
outputs/
  recon.md                # Phase 0 attacker recon + coupling hypothesis
  feynman-pass1.md        # Pass 1 Feynman findings
  state-pass2.md          # Pass 2 State Inconsistency findings
  feynman-pass3.md        # Pass 3 targeted Feynman (if needed)
  state-pass4.md          # Pass 4 targeted State (if needed)
  nemesis-verified.md     # Final consolidated + verified report
```

### 5. Resume if interrupted

```
nemesis audit --continue    # Picks up from the last completed pass
```

---

## All Commands

| Command | What It Does |
|---------|--------------|
| `nemesis audit` | Full iterative audit until convergence |
| `nemesis audit --pass1` | Pass 1 only — full Feynman Agent |
| `nemesis audit --pass2` | Pass 2 only — State Inconsistency on existing Pass 1 output |
| `nemesis audit --continue` | Resume from where the last pass left off |
| `nemesis audit --contract [name]` | Full audit scoped to a single contract |
| `feynman audit` | Run the Feynman Agent standalone |
| `state audit` | Run the State Inconsistency Agent standalone |

---

## Architecture

```
.antigravity/
  agents/
    nemesis.md            # Orchestrator — runs the iterative loop
    feynman.md            # First-principles logic bug finder
    state.md              # Coupled state desync detector
  AGENTS.md               # Architecture overview and migration notes

outputs/                  # Audit findings (gitignored or committed per preference)
```

### Original Claude Code skills (preserved for reference)
```
.claude/
  skills/
    nemesis-auditor/SKILL.md
    feynman-auditor/SKILL.md
    state-inconsistency-auditor/SKILL.md
```

---

## Finding Format

Each finding in the final report includes:

```markdown
### Finding NEM-001: [Title]
**Severity:** CRITICAL | HIGH | MEDIUM | LOW
**Source:** Feynman-only | State-only | Cross-feed Pass N -> Pass M

**Root Cause:** [What is broken and why]
**Trigger Sequence:**
1. [Step-by-step to reproduce]

**Impact:** [What goes wrong -- fund loss, locked state, etc.]
**Fix:** [Minimal code change]
**Verification:** Code trace | PoC sequence | Both
```

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
