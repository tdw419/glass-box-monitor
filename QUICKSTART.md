# Glass-Box: Bidirectional Agent Memory System

> **The Anti-Black Box**: Transform agents from amnesic to cumulatively intelligent

## Quick Start

```bash
# Clone
git clone https://github.com/tdw419/glass-box-monitor.git
cd glass-box-monitor

# Install scripts
cp scripts/glass-* ~/.openclaw/workspace/scripts/  # or ~/bin/
chmod +x ~/.openclaw/workspace/scripts/glass-*

# Initialize logs
mkdir -p ~/.openclaw/workspace/logs
touch ~/.openclaw/workspace/logs/glass-box-{hypotheses,decisions}.jsonl

# Try the demo
glass-demo
```

## What is Glass-Box?

Glass-Box solves the **agent amnesia problem**: agents forget everything between sessions.

Traditional logging is write-only: Agent → Logs → Human reads
Glass-Box is bidirectional: Agent ↔ Logs ↔ Agent queries

**NEW:** Now includes **LM Studio integration** for 5x faster code generation!

### Core Capabilities

1. **LM Studio Integration** - Fast local AI (2-3s vs 10-15s cloud) ⚡
2. **Hypothesis Tracking** - Remember what you tried and what worked
3. **Decision Logging** - Remember WHY you made choices
4. **Session Resumption** - Pick up where you left off
5. **Pattern Detection** - Learn from failure patterns

## Example Usage

### Track an Experiment
```bash
glass-box hypothesis "wfi instruction causes SIGILL"
glass-box test "Remove wfi, use plain loop"
glass-box result "✓ Works - no crash"

# Later, when stuck:
glass-box query "SIGILL"
→ [hypothesis] wfi causes SIGILL
→ [test] Remove wfi
→ [result] ✓ Works
→ PATTERN: Don't use privileged instructions in user mode
```

### Remember Decisions
```bash
glass-decision decision "Use syscalls not MMIO"
glass-reason "Kernel in S-mode, MMIO unavailable"

# Next session:
glass-decision why "syscalls"
→ DECISION: Use syscalls not MMIO
   REASON: Kernel in S-mode, MMIO unavailable
```

### Resume Work
```bash
glass-summary
╔════════════════════════════════════════════════════════════╗
║          GLASS-BOX SESSION SUMMARY                         ║
╚════════════════════════════════════════════════════════════╝

📊 Recent Milestones:
  ✓ RV32 Linux kernel built
  ✓ INIT OK - Linux boots in QEMU

💡 Suggested Next Action:
  - Fix kernel entry point for GPU boot
```

## Documentation

- **README.md** - Complete usage guide
- **AGENT_WORKFLOW.md** - Cross-session learning examples
- **SKILL.md** - Agent-facing instructions
- **references/ux-patterns.md** - Architectural patterns

## Installation

### For OpenClaw
```bash
cp scripts/glass-* ~/.openclaw/workspace/scripts/
```

### For Gemini CLI
```bash
gemini skills install glass-box.skill
/skills reload
```

### Standalone
```bash
cp scripts/glass-* ~/bin/
```

## Why This Matters

| Without Glass-Box | With Glass-Box |
|-------------------|----------------|
| Repeat mistakes 3-4 times | Query memory → avoid |
| Ask user "where were we?" | `glass-summary` → full context |
| Lost reasoning after session | Persistent across sessions |
| Linear learning curve | Cumulative intelligence |

## License

MIT - Use freely in any agent system.

## Contributing

Contributions welcome! Key areas:
- Pattern detection algorithms
- Cross-agent memory sync
- Visualization (web UI)
- Knowledge graph export

---

**Remember:** The goal isn't just transparency - it's **agent self-improvement through memory**.
