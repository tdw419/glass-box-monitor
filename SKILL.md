---
name: glass-box
description: Implements the "Glass Box" UX pattern (Transparency, Stability, Pulse) for autonomous agents. Replaces the "Black Box" problem with high-signal activity logs, foldable thinking, and sub-agent proxy streaming.
---

# Glass Box (Architectural Edition)

This skill transforms "black box" agent operations into transparent, stable, and interactive experiences. It implements the "Activity Log" pattern to mitigate platform rate-limiting and perceived latency.

## Core Patterns

- **Live Activity Log**: Granular status updates (`LOG: [ACTING] ...`, `LOG: [SEARCHING] ...`).
- **Markdown-Safe Buffering**: Hold formatting tokens (`, **, [) until closed to prevent UI flicker.
- **Foldable Thinking**: Strict separation of internal monologue from user-facing action.
- **Proxy Streaming**: Forwarding events from sub-agents directly to the user channel.

## Implementation Guardrails

1. **Adaptive Throttling**: Aim for 250ms updates on high-quota channels (Telegram Drafts/Slack) and 1000ms on constrained channels (Discord/WhatsApp).
2. **Stability Protocol**: Never emit a "Thinking" token as a "Response" token. Reasoning must be finalized or wrapped in a Thinking block before disclosure.
3. **Milestone Delivery**: For tasks > 5 minutes, emit a checklist of completed/pending sub-tasks every 60 seconds.

## Commands

- **glass-box-task <task>**: Runs a simulated task with these patterns.
  ```bash
  node scripts/glass_box_task.cjs "Your Task Description"
  ```

## Reference

For detailed pattern descriptions and design principles, see [ux-patterns.md](references/ux-patterns.md).
