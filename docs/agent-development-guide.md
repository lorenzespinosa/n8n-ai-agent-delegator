# Agent Development Guide

## Adding a New Agent

1. Create `workflows/1xxx-agent-yourname.json` (pick next available 1xxx number)
2. Include these mandatory nodes:
   - Webhook trigger (receives command from orchestrator)
   - Command parser (Code node extracting action + parameters)
   - Action-specific logic
   - Confidence scorer (Code node outputting 0.0-1.0)
   - Human approval gate (IF node, required for write operations)
   - Response formatter (Code node with standard output schema)

3. Add to orchestrator routing:
   - Add intent keyword to 0000-orchestrator.json Switch node
   - Add Execute Workflow node pointing to your agent

4. Create sample payloads:
   - `payloads/command-yourname.json` (input)
   - Test both high and low confidence paths

## Standard Agent Response Schema

```json
{
  "agent": "1xxx-agent-name",
  "action": "action_performed",
  "confidence": 0.85,
  "result": {},
  "requires_approval": false,
  "execution_time_ms": 1500
}
```

## Naming Convention

| Tier | Range | Purpose |
|------|-------|---------|
| 0000 | 0000 only | Orchestrator (one per system) |
| 1xxx | 1000-1999 | Specialist agents |
| 9xxx | 9000-9999 | Infrastructure (monitor, logging) |
