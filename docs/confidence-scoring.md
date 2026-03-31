# Confidence Scoring System

## How It Works

Every agent output is scored by the 9000-monitor before reaching the user. The composite score determines routing.

## Composite Score Formula

```
confidence = (0.70 × agent_score) + (0.20 × classification_score) + (0.10 × time_penalty)
```

| Component | Weight | Source |
|-----------|--------|--------|
| Agent score | 70% | Self-reported by the agent based on result quality |
| Classification score | 20% | Orchestrator's intent classification confidence |
| Time penalty | 10% | Deduction for slow responses (>10s = -5%, >30s = -10%) |

## Routing Thresholds (Current: 2-Band)

| Score | Route | Action |
|-------|-------|--------|
| >= 0.70 | Direct output | Deliver to user |
| < 0.70 | Human review | Route to human review queue before delivery |

### Planned Enhancement: 4-Band Model

A more granular routing model is planned for a future release:

| Score | Route | Action |
|-------|-------|--------|
| >= 0.85 | Direct output | Deliver to user immediately |
| 0.70 - 0.84 | Output with caveat | Deliver with "AI-generated, please verify" note |
| 0.50 - 0.69 | Human review queue | Slack notification, human reviews before delivery |
| < 0.50 | Reject | Re-prompt or escalate to human |

## Write Operation Override

Regardless of confidence score, ALL write operations require human approval:
- Sending emails
- Creating calendar events
- Publishing content
- Modifying records

This is a hard gate — confidence scoring does not bypass it.

## Tuning

Adjust thresholds in the 9000-monitor workflow's Code node. The defaults above are conservative — increase the human review threshold (0.70) for higher-stakes environments like legal.
