# Glass-Box + OpenClaw Integration

## Overview

Glass-Box now integrates with **OpenClaw session history** to provide comprehensive memory across all agent interactions.

## Storage Locations

```
OpenClaw Sessions:
~/.openclaw/agents/main/sessions/
├── sessions.json              # Metadata (4.2MB)
└── *.jsonl                     # Session logs (311 sessions, 56MB)

Glass-Box Memory:
~/.openclaw/workspace/logs/
├── glass-box-hypotheses.jsonl   # Experimental memory
├── glass-box-decisions.jsonl    # Decision memory
└── progress.jsonl                # Milestone history
```

## Tools

### 1. glass-openclaw
```bash
# List recent sessions
glass-openclaw recent

# Extract insights (dry run)
glass-openclaw extract

# Sync to Glass-Box
glass-openclaw sync
```

### 2. glass-query (Unified Search)
```bash
# Search both Glass-Box + OpenClaw
glass-query "SIGILL"

# Returns:
# - Glass-Box hypotheses
# - Glass-Box decisions
# - OpenClaw session matches
```

### 3. glass-current
```bash
# Extract from current session
glass-current
```

## Memory Types

| Type | Location | Purpose |
|------|----------|---------|
| **Hypotheses** | `glass-box-hypotheses.jsonl` | Experimental memory (test → result) |
| **Decisions** | `glass-box-decisions.jsonl` | Reasoning memory (choice + why) |
| **Sessions** | `~/.openclaw/agents/main/sessions/` | Full conversation history |
| **Progress** | `progress.jsonl` | Milestone tracking |

## Usage Patterns

### Pattern 1: Session Resumption
```bash
# Start of session
glass-summary

# Output:
╔════════════════════════════════════════════════════════════╗
║          GLASS-BOX SESSION SUMMARY                         ║
╚════════════════════════════════════════════════════════════╝

📊 Recent Milestones:
  ✓ RV32 Linux kernel built
  ✓ INIT OK - Linux boots in QEMU

🧪 Last Hypotheses:
  [hypothesis] Kernel entry at 0xc0000000 won't work on GPU
  [result] LM Studio recommends OpenSBI

💡 Suggested Next Action:
  - Build trampoline with page tables
  - Map 0xc0000000 virtual → 0x0 physical
```

### Pattern 2: Unified Search
```bash
# Search everything
glass-query "kernel boot"

# Returns matches from:
# - Glass-Box hypotheses
# - Glass-Box decisions  
# - OpenClaw sessions
```

### Pattern 3: Automatic Sync
```bash
# Add to HEARTBEAT.md for auto-sync
echo "glass-openclaw sync" >> ~/.openclaw/workspace/HEARTBEAT.md

# Now syncs automatically during heartbeat checks
```

## Integration Architecture

```
┌─────────────────────────────────────────────────────┐
│                  AGENT WORKFLOW                     │
└────────────────────┬────────────────────────────────┘
                     │
         ┌───────────▼───────────┐
         │   glass-summary        │
         │   (Session Start)      │
         └───────────┬───────────┘
                     │
    ┌────────────────┼────────────────┐
    │                │                │
┌───▼────┐     ┌────▼─────┐    ┌────▼──────┐
│ Memory │     │Fast AI   │    │Cloud AI   │
│        │     │          │    │           │
│glass-  │     │lm-code   │    │(fallback) │
│query   │     │lm-ask    │    │           │
└───┬────┘     └────┬─────┘    └───────────┘
    │               │
    │    ┌──────────▼──────────┐
    │    │  GLASS-BOX MEMORY   │
    │    │  - hypotheses       │
    │    │  - decisions        │
    │    └──────────┬──────────┘
    │               │
    │    ┌──────────▼──────────┐
    │    │  OPENCLAW SESSIONS  │
    │    │  - 311 sessions     │
    │    │  - 56MB history     │
    │    └─────────────────────┘
    │
    └────► glass-openclaw sync
```

## Data Flow

### During Session
```
1. User asks question
2. Agent checks glass-summary (context)
3. Agent uses lm-ask/lm-code (fast iteration)
4. Agent tracks with glass-box hypothesis/test/result
5. Agent records with glass-decision decision/reason
6. Session ends
```

### Between Sessions
```
1. glass-openclaw sync runs (heartbeat)
2. Extracts insights from OpenClaw sessions
3. Appends to Glass-Box memory
4. Next session: glass-summary shows unified view
```

## Performance

| Operation | Time | Notes |
|-----------|------|-------|
| `glass-summary` | Instant | Reads 3 files |
| `glass-query` | 1-2s | Searches 2 JSONL + sessions |
| `glass-openclaw recent` | 1s | Lists recent sessions |
| `glass-openclaw sync` | 5-10s | Extracts from recent sessions |

## Storage Growth

```
Current (3/20/2026):
- OpenClaw sessions: 56MB (311 sessions)
- Glass-Box memory: 2.4KB (hypotheses + decisions)

Projected (1 year):
- OpenClaw: ~2GB (~12,000 sessions)
- Glass-Box: ~5MB (distilled insights)

Ratio: Glass-Box is 0.25% of raw session size
```

## Scripts Reference

| Script | Purpose |
|--------|---------|
| `glass-box` | Hypothesis tracking |
| `glass-decision` | Decision logging |
| `glass-summary` | Session resumption |
| `glass-query` | Unified search |
| `glass-openclaw` | OpenClaw integration |
| `glass-current` | Current session extraction |
| `lm-code` | Fast code generation |
| `lm-ask` | Fast Q&A |

## Example Session

```bash
$ glass-summary

╔════════════════════════════════════════════════════════════╗
║          GLASS-BOX SESSION SUMMARY                         ║
╚════════════════════════════════════════════════════════════╝

📊 Recent Milestones:
  ✓ RV32 Linux kernel built
  ✓ INIT OK - Linux boots in QEMU
  ✓ Kernel loads on GPU

🧪 Last Hypotheses:
  [hypothesis] Kernel entry at 0xc0000000 won't work on GPU
  [test] Asked LM Studio about OpenSBI vs PHYS_OFFSET
  [result] LM Studio recommends OpenSBI

💡 Suggested Next Action:
  - Build trampoline with page tables
  - Map 0xc0000000 virtual → 0x0 physical
  - Test on GPU

$ lm-ask "How to build trampoline for RISC-V?" (2s)
$ lm-code "Generate trampoline assembly" 800 (2s)
$ glass-box test "Try trampoline approach"
# ... test ...
$ glass-box result "✓ Works!"
$ glass-decision decision "Use trampoline for kernel boot"
$ glass-reason "Maps virtual to physical addresses"
```

## Privacy

- **All data local** - No cloud sync
- **User controls** - Choose what to sync
- **Delete anytime** - `rm ~/.openclaw/workspace/logs/*.jsonl`

## Future Enhancements

1. **Semantic search** - Vector embeddings for better matching
2. **Auto-extraction** - Parse sessions for patterns automatically
3. **Topic modeling** - Identify recurring themes
4. **Cross-agent sync** - Share patterns between agents
5. **Web UI** - Visualize session history and memory

---

**Glass-Box now has complete memory across all agent interactions!** 🧠
