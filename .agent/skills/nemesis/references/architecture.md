# Nemesis — Agent Architecture Reference

This document describes the three-skill architecture of the Nemesis auditor.

## Skills

| Skill | Path | Role |
|-------|------|------|
| `nemesis` | `.agent/skills/nemesis/SKILL.md` | Orchestrator — runs the iterative loop, manages passes, writes final report |
| `feynman` | `.agent/skills/feynman/SKILL.md` | Sub-skill — first-principles logic bug finder |
| `state` | `.agent/skills/state/SKILL.md` | Sub-skill — coupled state desync detector |

## Loop

```
Phase 0: Recon → outputs/recon.md
Pass 1:  Feynman (full) → outputs/feynman-pass1.md
Pass 2:  State (full, enriched by Pass 1) → outputs/state-pass2.md
Pass 3:  Feynman (targeted, enriched by Pass 2) → outputs/feynman-pass3.md
Pass 4:  State (targeted, enriched by Pass 3) → outputs/state-pass4.md
... until convergence (max 6 passes)
Final:   outputs/nemesis-verified.md
```

## Trigger Phrases

| What you type | What happens |
|---------------|--------------|
| `nemesis audit` | Full iterative audit |
| `nemesis audit --pass1` | Feynman only |
| `nemesis audit --pass2` | State only (requires pass1 output) |
| `nemesis audit --contract [Name]` | Scoped to one contract/module |
| `nemesis audit --continue` | Resume from last completed pass |
| `feynman audit` | Feynman skill standalone |
| `state audit` | State skill standalone |

## Output Files

```
outputs/
  recon.md                # Phase 0 attacker recon
  feynman-pass1.md        # Pass 1 Feynman findings
  state-pass2.md          # Pass 2 State findings
  feynman-pass3.md        # Pass 3 targeted Feynman (if needed)
  state-pass4.md          # Pass 4 targeted State (if needed)
  nemesis-verified.md     # Final consolidated + verified report
```

## Scope Path

All skills are placed under `.agent/skills/` (workspace scope), making them available only within the project that contains this folder. To make them available globally across all projects, copy to `~/.gemini/antigravity/skills/`.

## Original Claude Code Version

The original Claude Code skills are preserved in `.claude/skills/` for reference and cross-compatibility.
