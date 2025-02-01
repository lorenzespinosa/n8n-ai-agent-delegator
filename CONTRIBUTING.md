# Contributing to n8n-ai-agent-delegator

Thank you for contributing. This repo implements production-quality multi-agent AI task delegation patterns in n8n — contributions must meet the same quality bar.

## Before You Submit

- [ ] Workflow JSON has `active: false`
- [ ] `meta.instanceId` is stripped from workflow JSON
- [ ] Root-level `id` is stripped from workflow JSON
- [ ] No API keys, tokens, or real credentials in any file
- [ ] All sample data uses fictional values
- [ ] Agent workflows include confidence scoring on outputs
- [ ] Human-in-the-loop gate exists before any write operations (send email, create event, publish)
- [ ] Node names follow the `TIER-CATEGORY - Action` convention (0000 = orchestrator, 1xxx = agents, 9xxx = monitoring)

## Architecture Conventions

- **0000-orchestrator** — Central controller, intent classification, routing
- **1000-agent-email** — Email operations (draft, send, summarize)
- **1001-agent-calendar** — Calendar operations (check, schedule, reschedule)
- **1002-agent-research** — Web search, document Q&A
- **1003-agent-content** — Content generation (drafts, social, summaries)
- **9000-monitor** — Confidence scoring, audit logging, performance tracking

## Workflow JSON Standards

- Use the `Code` node — not the deprecated `Function` node
- Pin `typeVersion` to the version you tested against
- Set `active: false` in the root of the JSON
- Strip instance-specific fields: `meta.instanceId`, root `id`
- Error handling: import patterns from [n8n-error-handling-pattern](https://github.com/lorenzespinosa/n8n-error-handling-pattern)

## Pull Request Process

1. Fork the repo and create a branch: `feat/your-agent-name`
2. Add or update workflow JSON in `workflows/`
3. Add matching sample payloads in `payloads/` (input commands + output responses)
4. Update `CHANGELOG.md` with your addition
5. Run the pre-submit checklist above
6. Open a PR — CI validates JSON syntax and checks for credentials

## Reporting Issues

Use the issue templates provided in `.github/ISSUE_TEMPLATE/`.
