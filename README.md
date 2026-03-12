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

An iterative deep-logic security audit skill system for [Antigravity IDE](https://antigravity.google). Nemesis combines two complementary audit methodologies in an alternating feedback loop to find bugs that neither can catch alone. **Language-agnostic** — works on Solidity, Move, Rust, Go, C++, Python, TypeScript, or any codebase.

> Fork of [nemesis-auditor](https://github.com/0xiehnnkta/nemesis-auditor) adapted for Antigravity.
> The original Claude Code version (`.claude/` folder) is preserved for cross-compatibility.

---

## How It Works

Nemesis runs an **iterative back-and-forth loop** between three skills:

| Skill | Role | What It Finds |
|-------|------|---------------|
| `nemesis` | Orchestrator | Runs the loop, manages passes, writes final report |
| `feynman` | Sub-skill | Business logic bugs via first-principles questioning |
| `state` | Sub-skill | Coupled state desync bugs — maps every mutation path |

The loop alternates Feynman → State → Feynman → State until **convergence** (no new findings), max 6 passes.

---

## Requirements

- [Antigravity IDE](https://antigravity.google) installed and configured
- A codebase to audit

---

## Installation

### Option A: Copy into your project (recommended)

```bash
# Clone this repo
git clone https://github.com/rusrio/nemesis-auditor-antigravity.git

# Copy the .agent folder into your project
cp -r nemesis-auditor-antigravity/.agent /path/to/your-project/

# Create the outputs directory
mkdir -p /path/to/your-project/outputs

# Remove the cloned repo
rm -rf nemesis-auditor-antigravity
```

### Option B: Install globally (available in all your projects)

```bash
git clone https://github.com/rusrio/nemesis-auditor-antigravity.git
cp -r nemesis-auditor-antigravity/.agent/skills/* ~/.gemini/antigravity/skills/
rm -rf nemesis-auditor-antigravity
```

### Option C: Run from this repo

```bash
git clone https://github.com/rusrio/nemesis-auditor-antigravity.git
cd nemesis-auditor-antigravity

# Symlink your codebase into this directory
ln -s /path/to/your/contracts ./contracts

# Open in Antigravity
antigravity .
```

---

## Usage

### 1. Open your project in Antigravity

```bash
cd /path/to/your-project   # must contain .agent/skills/
antigravity .
```

### 2. Launch the audit

Type any of these in the Antigravity chat:

```
nemesis audit                           # Full iterative audit (recommended)
nemesis audit --contract MyToken        # Audit a single contract
nemesis audit --pass1                   # Only run the Feynman skill
nemesis audit --pass2                   # Only run the State skill
nemesis audit --continue                # Resume if interrupted
```

### 3. What happens

1. **Phase 0 — Recon:** Attacker mindset scan, builds hit list and coupling hypothesis → `outputs/recon.md`
2. **Pass 1 — Feynman:** Questions every function line-by-line. Produces findings, suspects, exposed assumptions → `outputs/feynman-pass1.md`
3. **Pass 2 — State:** Maps every coupled state pair, traces mutation paths, finds desync gaps enriched by Pass 1 suspects → `outputs/state-pass2.md`
4. **Pass 3+ — Iterative:** Feynman re-interrogates State's gaps. State re-checks Feynman's new findings. Alternates until convergence.
5. **Final — Consolidation:** Deduplicates, verifies all Critical/High/Medium findings → `outputs/nemesis-verified.md`

### 4. Read the results

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
nemesis audit --continue
```

---

## All Commands

| Command | What It Does |
|---------|--------------|
| `nemesis audit` | Full iterative audit until convergence |
| `nemesis audit --pass1` | Pass 1 only — full Feynman skill |
| `nemesis audit --pass2` | Pass 2 only — State skill on existing Pass 1 output |
| `nemesis audit --continue` | Resume from last completed pass |
| `nemesis audit --contract [name]` | Full audit scoped to a single contract |
| `feynman audit` | Run Feynman skill standalone |
| `state audit` | Run State skill standalone |

---

## Architecture

```
.agent/
  skills/
    nemesis/
      SKILL.md              # Orchestrator — runs the iterative loop
      references/
        architecture.md     # Architecture reference
    feynman/
      SKILL.md              # First-principles logic bug finder
    state/
      SKILL.md              # Coupled state desync detector

outputs/                    # Audit findings per pass
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

```markdown
### Finding NEM-001: [Title]
**Severity:** CRITICAL | HIGH | MEDIUM | LOW
**Source:** Feynman-only | State-only | Cross-feed Pass N -> Pass M

**Root Cause:** [What is broken and why]
**Trigger Sequence:**
1. [Step-by-step to reproduce]

**Impact:** [Fund loss, locked state, incorrect accounting, etc.]
**Fix:** [Minimal code change]
**Verification:** Code trace | PoC sequence | Both
```

---

## Language Support

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
