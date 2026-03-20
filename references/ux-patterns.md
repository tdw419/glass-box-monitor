# Progressive Disclosure UX Patterns

These patterns are designed to solve the "Black Box" problem in autonomous agent execution, where the user is left waiting for a long time without knowing what the agent is doing.

## 1. Live Log (Pulse Indicators)
Instead of waiting for the final output, the agent should emit periodic status updates.
- **Pattern:** `LOG: [STATUS] <message>`
- **Purpose:** Shows that the agent is still alive and provides a "heartbeat" of activity.
- **Example:** `LOG: [STATUS] Running unit tests for user-auth.ts...`

## 2. Foldable Thinking (Internal Monologue)
Separate the agent's internal reasoning from the final result. 
- **Pattern:** `THINKING: <monologue>`
- **Purpose:** Provides transparency into the agent's decision-making process without cluttering the final response.
- **UI Render:** Gemini CLI renders this in a `<details>`/`<summary>` block.

## 3. Milestone Commits
Divide long tasks into sub-tasks and signal their completion immediately.
- **Pattern:** `MILESTONE: <achievement>`
- **Purpose:** Gives the user a sense of progress and allows for early intervention if a sub-task fails or takes an unexpected turn.
- **Example:** `MILESTONE: Configuration files validated.`

## 4. Delta Streaming
If the task involves generating large amounts of data or code, stream the changes in small, logical chunks (deltas) rather than waiting for the entire block to be ready.
