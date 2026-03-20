# Glass Box (Architectural Edition)

> **The Anti-Black Box**: A bidirectional memory substrate for autonomous agents.

Glass Box solves the fundamental problem of agent memory: **agents forget everything between sessions**. It transforms the "Black Box" problem into a transparent, queryable, and self-improving system.

## 🎯 What Makes This Different

Most agent logging is **write-only** - the agent writes logs for humans to read. Glass Box is **bidirectional**:

- **Agent can READ** → Query past hypotheses, decisions, patterns
- **Agent can WRITE** → Track reasoning, experiments, results
- **Agent can LEARN** → Detect failure patterns, remember successes

```
Traditional Logging:  Agent → Logs → Human reads
Glass Box:            Agent ↔ Logs ↔ Agent queries
```

## ✨ Core Capabilities

### 1. Hypothesis Tracking (Experimental Memory)
Track what you tried, what worked, what failed:
```bash
glass-box hypothesis "wfi instruction causes SIGILL in user mode"
glass-box test "Remove wfi, use plain infinite loop"
glass-box result "✓ Works - no crash"

# Later, when stuck again:
glass-box query "SIGILL"
→ 16:36 [hypothesis] wfi causes SIGILL
→ 16:37 [test] Remove wfi
→ 16:43 [result] ✓ Works
→ PATTERN: Don't use privileged instructions in user mode
```

### 2. Decision Logging (Reasoning Memory)
Remember WHY you made choices:
```bash
glass-decision decision "Use syscalls not MMIO for init"
glass-reason "Kernel runs in S-mode, MMIO not available"

# Next session, when revisiting:
glass-decision why "syscalls"
→ DECISION: Use syscalls not MMIO for init
   REASON: Kernel runs in S-mode, MMIO not available
```

### 3. Session Resumption (Context Memory)
Pick up where you left off:
```bash
glass-summary
╔════════════════════════════════════════════════════════════╗
║          GLASS-BOX SESSION SUMMARY                         ║
╚════════════════════════════════════════════════════════════╝

📊 Recent Milestones:
  ✓ RV32 Linux kernel built
  ✓ INIT OK - Linux boots in QEMU
  ✓ Kernel loads on GPU

🧪 Last Hypotheses:
  [hypothesis] Entry point mismatch (0xc0000000 vs 0x00000000)
  [test] Need OpenSBI or direct boot

💡 Suggested Next Action:
  - Fix kernel entry point for GPU boot
  - Add device tree or OpenSBI firmware
```

### 4. Pattern Detection (Learning Memory)
Detect when you're repeating mistakes:
```bash
glass-box query "fail" | grep SIGILL
→ Failed 3 times with SIGILL
→ SUGGESTION: Check instruction privilege levels
```

## 🚀 Quick Start

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/glass-box.git
cd glass-box

# Install as Gemini CLI skill
gemini skills install glass-box.skill --scope user

# Add scripts to PATH
export PATH="$PATH:~/.openclaw/workspace/scripts"

# Initialize logs
mkdir -p ~/.openclaw/workspace/logs
touch ~/.openclaw/workspace/logs/glass-box-{hypotheses,decisions}.jsonl
```

### Basic Usage

```bash
# 1. Track an experiment
glass-box hypothesis "Approach X will solve problem Y"
glass-box test "Implementing X..."
glass-box result "✓ Works" or "✗ Failed because Z"

# 2. Record a decision
glass-decision decision "Choose framework A over B"
glass-reason "A has better GPU support, critical for our use case"

# 3. Query your memory
glass-box query "GPU"           # What did I try with GPU?
glass-decision why "framework"  # Why did I choose this?

# 4. Resume work
glass-summary                   # Where was I?
```

## 📐 Architecture

```
glass-box/
├── SKILL.md                           # Agent instructions
├── scripts/
│   ├── glass-box                      # Hypothesis tracker
│   ├── glass-decision                 # Decision logger
│   ├── glass-summary                  # Session summarizer
│   ├── progress-log.sh                # User-visible logging
│   └── progress_query.sh              # Log query tool
├── references/
│   └── ux-patterns.md                 # Architectural patterns
└── logs/                              # JSONL memory store
    ├── glass-box-hypotheses.jsonl     # Hypothesis tracking
    ├── glass-box-decisions.jsonl      # Decision memory
    └── progress.jsonl                 # Milestone history
```

## 🧠 Memory Types

| Type | Purpose | Query Example |
|------|---------|---------------|
| **Hypothesis** | Experimental memory | `glass-box query "SIGILL"` |
| **Decision** | Reasoning memory | `glass-decision why "syscalls"` |
| **Milestone** | Progress memory | `progress_query.sh "milestone"` |
| **Pattern** | Learning memory | Detect repeated failures |

## 📊 Real-World Example

### Session 1: Debugging RISC-V Boot
```bash
glass-box hypothesis "Alpine kernel is RV64, need RV32"
glass-box test "Build kernel with rv32_defconfig"
glass-box result "✓ Kernel compiles (29MB)"

glass-decision decision "Verify in QEMU before GPU"
glass-reason "Faster iteration cycle"

glass-box test "Boot in QEMU"
glass-box result "✓ INIT OK - Linux boots!"

glass-box test "Load on GPU"
glass-box result "✗ No output - entry point mismatch"
```

### Session 2: Resuming Work
```bash
glass-summary
→ Shows: Last work on GPU boot, entry mismatch
→ I remember: Need to fix 0xc0000000 vs 0x00000000

glass-box query "entry"
→ [hypothesis] Entry point mismatch
→ [result] ✗ No output

glass-decision why "QEMU"
→ "Faster iteration cycle"
→ I remember: Strategy is verify-then-port

# Continue with informed context...
```

## 🎨 Integration Patterns

### For OpenClaw Agents
```bash
# Add to workspace scripts
cp scripts/* ~/.openclaw/workspace/scripts/

# Use in MEMORY.md
**Glass-Box Memory:**
- Hypothesis: `glass-box hypothesis/test/result`
- Decisions: `glass-decision decision/reason/why`
- Summary: `glass-summary` at session start
```

### For Claude Code / Gemini CLI
```bash
# Install as skill
gemini skills install glass-box.skill

# Reload to activate
/skills reload
```

### For Custom Agents
```python
import json
from datetime import datetime
from pathlib import Path

class GlassBox:
    def __init__(self, log_dir="~/.openclaw/workspace/logs"):
        self.log_dir = Path(log_dir).expanduser()
        
    def hypothesis(self, prediction):
        self._log("hypotheses", "hypothesis", prediction)
        
    def test(self, action):
        self._log("hypotheses", "test", action)
        
    def result(self, outcome):
        self._log("hypotheses", "result", outcome)
        
    def query(self, keyword):
        return self._search("hypotheses", keyword)
        
    def _log(self, file, type, msg):
        log_file = self.log_dir / f"glass-box-{file}.jsonl"
        entry = {
            "ts": datetime.now().isoformat(),
            "type": type,
            "msg": msg
        }
        with open(log_file, "a") as f:
            f.write(json.dumps(entry) + "\n")
```

## 🔬 Advanced Patterns

### Pattern Detection
Detect when you're repeating mistakes:
```bash
# Count failures
glass-box query "fail" | wc -l
→ 7 failures recorded

# Group by type
glass-box query "SIGILL" | grep result
→ ✗ Failed with SIGILL (3 times)
→ SUGGESTION: Check privilege levels
```

### Success Templates
Extract patterns from successes:
```bash
# What worked for booting?
glass-box query "boot" | grep "✓"
→ ✓ Use rv32_defconfig
→ ✓ Test in QEMU first
→ ✓ Use syscalls not MMIO

# Apply to new problem
# 1. Check architecture match
# 2. Verify in emulator
# 3. Use proper ABI
```

### Cross-Session Learning
Build knowledge over time:
```bash
# Session 1: Learn about SIGILL
glass-box result "✓ SIGILL fixed by removing wfi"

# Session 5: New SIGILL appears
glass-box query "SIGILL"
→ Remember: Check for privileged instructions
glass-box hypothesis "Same issue - privileged instruction?"
glass-box test "Remove wfi again"
glass-box result "✓ Works!"
```

## 📈 Metrics

Track your problem-solving efficiency:
```bash
# Success rate
glass-box query "result" | grep "✓" | wc -l
→ 12 successes

# Failure rate
glass-box query "result" | grep "✗" | wc -l
→ 5 failures

# Improvement over time
→ Week 1: 40% success rate
→ Week 2: 65% success rate
→ Week 3: 80% success rate
```

## 🤝 Contributing

Contributions welcome! Key areas:
- **Pattern detection algorithms** - Auto-suggest based on history
- **Cross-agent sync** - Share memory between agents
- **Visualization** - Web UI for hypothesis/decision trees
- **Export** - Convert to knowledge graphs, documentation

## 📚 Further Reading

- `references/ux-patterns.md` - Architectural patterns from OpenClaw analysis
- `SKILL.md` - Agent-facing instructions
- [OpenClaw Architecture Analysis](link) - The paper that inspired this

## 📄 License

MIT License - Use freely in any agent system.

---

**Remember:** The goal isn't just transparency - it's **agent self-improvement through memory**. Glass Box makes your agent smarter with every session.
