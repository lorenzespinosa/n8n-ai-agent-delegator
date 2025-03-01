# Multi-Agent Orchestration in Make (Integromat)

## Architecture Pattern

The n8n agent delegator translates to Make using **linked scenarios** as the delegation mechanism. The orchestrator is one scenario; each agent is a separate scenario invoked via Make's internal webhook or scenario-call modules.

## Orchestrator Routing Logic

### Intent Classification

1. **Webhook trigger** receives the natural-language command (same payload structure).
2. **HTTP module** calls OpenAI for intent classification — identical API call, just configured in Make's HTTP module with JSON body.
3. **Router module** replaces the n8n Switch node. Each route has a filter condition matching the classified intent.

### Router Configuration

Make's Router evaluates filters top-to-bottom, first match wins:

- **Route 1 — Email:** Filter: `intent` equals `email` → calls email agent scenario
- **Route 2 — Calendar:** Filter: `intent` equals `calendar` → calls calendar agent scenario
- **Route 3 — Research:** Filter: `intent` equals `research` → calls research agent scenario
- **Route 4 — Content:** Filter: `intent` equals `content` → calls content agent scenario
- **Fallback route:** No filter (catches everything else) → human review path

### Agent Delegation via Scenario Calls

Two approaches for calling sub-scenarios:

**Option A: Internal Webhooks**
- Each agent scenario starts with a Custom Webhook trigger.
- The orchestrator's Router routes hit an HTTP module that POSTs to the agent's webhook URL.
- Limitation: asynchronous — the orchestrator won't wait for the agent's response inline.

**Option B: Make API + Run Scenario**
- Use Make's own API (`POST /scenarios/{id}/run`) to trigger the agent scenario.
- Still async, but you can poll for completion or use a webhook callback.
- Better for audit trails since you get an execution ID back.

**Recommended:** Option A with a **webhook callback pattern**. The agent scenario POSTs its result back to a second webhook on the orchestrator, which then aggregates and responds.

## Confidence Gating

After the agent callback returns:

1. **Filter module** checks `confidence < 0.7`.
2. True path → HTTP module sends Slack alert (same webhook payload as n8n version).
3. Both paths → HTTP module appends row to Google Sheets audit log.

## Human Approval for Write Operations

Make doesn't have a native "wait for approval" node. Options:

- **Approval webhook pattern:** Agent scenario pauses (ends), sends Slack message with approve/reject buttons pointing to a new webhook. A second scenario picks up the approval and completes the write.
- **Make's built-in "Break" + resume:** Use error handling with Break directive, then resume manually from the execution log. Less automated, but simpler.

## Key Differences from n8n

| Concern | n8n | Make |
|---|---|---|
| Sub-workflow call | Execute Workflow (sync) | HTTP to webhook (async) |
| Approval gate | Wait node / Form trigger | Webhook callback pattern |
| Code execution | Code node (full JS) | JavaScript/Python module (limited) |
| Variable passing | Expression `{{ }}` | Variable mapping UI |
| Error retry | Built-in retry with backoff | Error handler → retry route |

## Scenario Structure

```
Orchestrator Scenario
├── Webhook (trigger)
├── HTTP → OpenAI (classify)
├── Router
│   ├── Route: email → HTTP → agent-email webhook
│   ├── Route: calendar → HTTP → agent-calendar webhook
│   ├── Route: research → HTTP → agent-research webhook
│   └── Fallback → HTTP → Slack (unknown intent)
└── (Callback webhook receives agent response)
    ├── Filter: confidence < 0.7 → Slack alert
    └── HTTP → Google Sheets (audit log)

Agent Scenario (each)
├── Custom Webhook (trigger)
├── Router (by action sub-type)
│   ├── Route: action A → HTTP → API call
│   ├── Route: action B → HTTP → approval webhook → API call
│   └── Fallback → error response
├── Set Variable (format response + confidence)
└── HTTP → callback to orchestrator webhook
```

## Limitations

- No synchronous sub-scenario execution — requires the callback webhook pattern.
- JavaScript module in Make has restricted `require()` — no external npm packages.
- Router filter expressions are less flexible than n8n's Switch with Code nodes.
- Execution history is per-scenario, not unified — harder to trace a full command lifecycle without the audit sheet.
