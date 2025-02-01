# Changelog

All notable changes to n8n-ai-agent-delegator are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html)

## [Unreleased]

## [0.1.0] - 2025-02-01

### Added
- Initial repository scaffold with MIT license, CONTRIBUTING.md, and issue templates
- GitHub Actions CI for JSON validation (credential leak detection, active flag check)
- Stale issue bot configuration (auto-close after 30 days)
- `.gitignore` blocking credential files and n8n instance metadata
- README shell with multi-agent architecture diagram
- Tiered naming convention: 0000 (orchestrator), 1xxx (agents), 9000 (monitoring)
- Error handling referenced from n8n-error-handling-pattern
- Legal ops integration referenced from n8n-legal-ops-templates
- Repository structure: `workflows/`, `payloads/`, `docs/`
