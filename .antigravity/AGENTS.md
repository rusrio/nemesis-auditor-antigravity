# Nemesis — Antigravity Agent Architecture

This directory contains the Antigravity adaptation of the Nemesis security auditor.

## Agents

| Agent | File | Role |
|-------|------|------|
| `nemesis` | `agents/nemesis.md` | Orchestrator — runs the iterative loop, manages passes, writes final report |
| `feynman` | `agents/feynman.md` | Sub-agent — first-principles logic bug finder |
| `state` | `agents/state.md` | Sub-agent — coupled state desync detector |

## How to Use

Open your project in Antigravity, then:

```
nemesis audit          → full iterative audit (recommended)
nemesis audit --pass1  → feynman only
nemesis audit --pass2  → state only (requires existing pass1 output)
nemesis audit --contract [Name]  → scoped to one contract/module
nemesis audit --continue         → resume from last completed pass
```

## Output Files

All findings are saved to `outputs/` in your project root:

```
outputs/
  recon.md                # Phase 0 attacker recon
  feynman-pass1.md        # Pass 1 Feynman findings
  state-pass2.md          # Pass 2 State findings
  feynman-pass3.md        # Pass 3 targeted Feynman (if needed)
  state-pass4.md          # Pass 4 targeted State (if needed)
  nemesis-verified.md     # Final consolidated + verified report
```

## Architecture vs Original (Claude Code)

| Claude Code | Antigravity |
|------------|-------------|
| `.claude/skills/nemesis-auditor/SKILL.md` | `.antigravity/agents/nemesis.md` |
| `.claude/skills/feynman-auditor/SKILL.md` | `.antigravity/agents/feynman.md` |
| `.claude/skills/state-inconsistency-auditor/SKILL.md` | `.antigravity/agents/state.md` |
| `/nemesis` slash command | `nemesis audit` in chat |
| `.audit/findings/` | `outputs/` |
| Claude Code loads skills via `Load: path/to/SKILL.md` | Nemesis agent delegates via `run_agent: feynman` / `run_agent: state` |

## Notes

- The original `.claude/` folder is kept intact for reference and cross-compatibility.
- The logic, methodology, and prompts are identical — only the runtime wiring changes.
- Language-agnostic: works on Solidity, Move, Rust, Go, C++, Python, TypeScript.
