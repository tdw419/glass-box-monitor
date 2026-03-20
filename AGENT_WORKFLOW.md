# Glass-Box Agent Workflow Example

This shows how an agent uses Glass Box across multiple sessions to solve a complex problem.

## Session 1: Initial Investigation

```bash
# Agent starts work on a new problem
glass-box hypothesis "RISC-V kernel won't boot on GPU - architecture mismatch"
glass-box test "Check if kernel is RV64 vs RV32"
glass-box result "✓ Alpine kernel is RV64, shader expects RV32"

glass-decision decision "Build RV32 kernel from source"
glass-reason "Shader only supports RV32IMA instruction set"

glass-box test "make ARCH=riscv rv32_defconfig"
glass-box result "✓ Kernel config generated"

glass-box test "make -j\$(nproc)"
glass-box result "✓ Kernel compiled (29MB Image)"

glass-box test "Test boot in QEMU first"
glass-box result "✓ INIT OK - Linux boots in QEMU"

glass-box test "Load kernel on GPU"
glass-box result "✗ No output - entry point mismatch"

glass-summary > session1-recap.txt
```

## Session 2: Resuming Work (Next Day)

```bash
# Agent starts new session, checks memory
glass-summary
→ Shows: Last work on GPU boot, entry mismatch
→ I remember: Need to fix 0xc0000000 vs 0x00000000

glass-box query "entry"
→ [hypothesis] Entry point mismatch
→ [result] ✗ No output

glass-decision why "QEMU"
→ "Faster iteration cycle"
→ I remember: Strategy is verify-then-port

# Agent continues with context restored
glass-box hypothesis "Kernel linked at 0xc0000000, GPU expects 0x00000000"
glass-box test "Check vmlinux entry point"
glass-box result "✓ Confirmed: entry at 0xc0000000"

glass-decision decision "Add OpenSBI firmware layer"
glass-reason "OpenSBI handles virtual-to-physical translation"

# ... work continues ...
```

## Session 3: Pattern Recognition

```bash
# Agent encounters similar issue with different kernel
glass-box hypothesis "New kernel has same entry point issue"

# Agent queries memory for pattern
glass-box query "entry"
→ Session 1: Entry mismatch with Alpine kernel
→ Session 2: Fixed with OpenSBI
→ PATTERN DETECTED: Always check entry point first!

# Agent applies learned pattern immediately
glass-box test "Check entry point before compiling"
glass-box result "✓ Entry at 0x80000000 - needs adjustment"

# Success comes faster due to memory
glass-box result "✓ Kernel boots on first try!"
```

## Key Benefits Demonstrated

1. **No Repeated Mistakes**: Agent remembers SIGILL fix from Session 1
2. **Context Preservation**: Session 2 starts with full context
3. **Pattern Learning**: Session 3 applies pattern from Sessions 1-2
4. **Faster Iteration**: Agent knows "verify in QEMU first" strategy

## Efficiency Metrics

```
Without Glass Box:
  Session 1: 4 hours (trial and error)
  Session 2: 3 hours (re-learning context)
  Session 3: 3 hours (repeating mistakes)
  Total: 10 hours

With Glass Box:
  Session 1: 4 hours (learning)
  Session 2: 2 hours (context restored)
  Session 3: 1 hour (pattern applied)
  Total: 7 hours (30% faster)
```

## The Learning Curve

```
Success Rate Over Time:
  Week 1: ████████░░ 40% (learning basics)
  Week 2: ████████████░░ 65% (remembering patterns)
  Week 3: ████████████████ 80% (applying learned solutions)
  Week 4: ██████████████████ 90% (pattern mastery)
```

## Memory Persistence

All data stored in simple JSONL files:
```
~/.openclaw/workspace/logs/
├── glass-box-hypotheses.jsonl    # 3 months of experiments
├── glass-box-decisions.jsonl     # 50+ architectural decisions
└── progress.jsonl                # 200+ milestones
```

**Query any point in history:**
```bash
glass-box query "SIGILL" --since="2026-01-01"
→ Shows all SIGILL debugging across 3 months
```

This is the power of bidirectional agent memory.
