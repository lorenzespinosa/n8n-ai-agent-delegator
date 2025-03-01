# Multi-Agent Orchestration in Zapier

## Architecture Pattern

Zapier's architecture maps less cleanly to the agent delegator pattern than n8n or Make. The key tools are **Paths** (conditional branching), **Code by Zapier** (JavaScript/Python), and **Webhooks by Zapier** (inter-Zap communication).

## Intent Classification with Paths

### Orchestrator Zap

1. **Trigger: Webhooks by Zapier (Catch Hook)** — receives the natural-language command payload.
2. **Action: Code by Zapier** — sends the command to OpenAI for classification.

```javascript
// Code by Zapier step
const response = await fetch('https://api.openai.com/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${inputData.openai_key}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    model: 'gpt-4o',
    messages: [{
      role: 'system',
      content: 'Classify intent: email, calendar, research, content, unknown. Return JSON.'
    }, {
      role: 'user',
      content: inputData.command
    }],
    temperature: 0.1,
    response_format: { type: 'json_object' }
  })
});
const data = await response.json();
const classification = JSON.parse(data.choices[0].message.content);
output = [{
  intent: classification.intent,
  confidence: classification.confidence,
  action: classification.action,
  command: inputData.command
}];
```

3. **Paths** — branch by classified intent.

### Path Configuration

Each Path has a condition rule:

- **Path A — Email:** `intent` exactly matches `email`
- **Path B — Calendar:** `intent` exactly matches `calendar`
- **Path C — Research:** `intent` exactly matches `research`
- **Path D — Content:** `intent` exactly matches `content`
- **Default Path:** catches `unknown` and anything else

### Agent Delegation

Within each Path, delegate to the agent Zap:

- **Webhooks by Zapier (POST)** — sends the command + classification to the agent Zap's catch hook URL.
- The agent Zap processes and POSTs results back to a second catch hook on the orchestrator.

Alternatively, for simpler cases, put the entire agent logic inline within the Path (works for 3-4 step agents, breaks down for complex ones).

## Agent Zap Pattern

Each agent is a separate Zap:

```
Agent Zap (e.g., Email Agent)
├── Trigger: Webhooks by Zapier (Catch Hook)
├── Code by Zapier: parse command, determine sub-action
├── Paths (by sub-action)
│   ├── Path: draft
│   │   └── Code by Zapier: call OpenAI for email draft
│   ├── Path: send
│   │   ├── Code by Zapier: format approval request
│   │   └── Webhooks by Zapier: POST to Slack for approval
│   └── Path: summarize
│       └── Code by Zapier: call OpenAI for summary
├── Code by Zapier: format response + confidence score
└── Webhooks by Zapier: POST result back to orchestrator
```

## Confidence Scoring

Add a Code by Zapier step after the agent callback:

```javascript
const confidence = parseFloat(inputData.confidence);
const threshold = 0.7;
output = [{
  confidence: confidence,
  below_threshold: confidence < threshold,
  status: confidence < threshold ? 'review_required' : 'completed'
}];
```

Then use a **Filter** or **Path** to route low-confidence results to Slack.

## Human Approval for Write Operations

Zapier has no native "wait for human approval" step. Workarounds:

1. **Slack + Delay pattern:** Send approval request to Slack, use a separate Zap triggered by Slack reaction/button that completes the write. Requires splitting into two Zaps.
2. **Email approval:** Use "Send Outbound Email" with approve/reject links pointing to catch hooks.
3. **Zapier Interfaces (Tables):** Create a review queue in Zapier Tables. A second Zap watches for status changes and completes approved actions.

## Limitations

Zapier is the weakest fit for this pattern. Key constraints:

| Limitation | Impact |
|---|---|
| **Paths max depth** | Paths cannot be nested. One level of branching per Zap. Complex routing requires chaining Zaps. |
| **Code by Zapier runtime** | 1-second execution limit on free plans, 30 seconds on paid. Complex AI calls may timeout. |
| **No synchronous sub-Zap calls** | Unlike n8n's Execute Workflow, there's no way to call a Zap and wait for its result inline. Always async via webhooks. |
| **Limited error handling** | No built-in retry with backoff. Must implement manually in Code steps or use Zapier's auto-replay (which retries the entire Zap). |
| **Variable passing between Paths** | Data from one Path doesn't flow to steps after the Paths block. Must use a Code step or Formatter to merge. |
| **Task consumption** | Each step in each Zap consumes a task. A full orchestrator + agent round-trip can consume 15-25 tasks per command. At scale, this gets expensive fast. |
| **No looping** | Can't iterate or retry within a Zap without Looping by Zapier (limited to simple array iteration, not retry logic). |

## When Zapier Works

- **Prototype/demo** of the agent pattern with 1-2 simple agents
- **Low-volume** usage (< 50 commands/day) where task consumption isn't a concern
- **Teams already on Zapier** who want a proof-of-concept before migrating to n8n

## When to Use n8n Instead

- More than 2 agents or complex sub-actions
- Need synchronous agent execution
- Need human-in-the-loop approval gates
- High volume (tasks add up fast in Zapier)
- Need full audit trail in a single execution log

## Recommended Approach

If you must build this in Zapier:

1. Keep agents simple — 3-4 steps max per agent Zap
2. Use Code by Zapier for all AI calls and data transformation
3. Use Zapier Tables as the human review queue
4. Log everything to Google Sheets (same audit pattern as n8n version)
5. Accept that the orchestrator-to-agent round-trip is async and design the UX accordingly (e.g., "Your request is being processed" immediate response, result delivered via Slack/email)
