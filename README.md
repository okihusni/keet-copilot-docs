# Keet Copilot — Shared Engineering Docs

This repository is the shared documentation and decision record for Keet Copilot. It is separate from the application source repository.

## Workstreams
- linux-audio-capture/ — Linux-specific audio capture documentation.
- windows-gemini-live/ — Windows-specific Gemini Live implementation documentation.
- collaboration/ — rules, roles, and review workflow.
- reviews/ — review records, evidence, decisions, and implementation reports.

Keep platform-specific instructions in their own workstream. Do not overwrite or merge Linux and Windows assumptions.

## Collaboration
Start with [Collaboration Protocol](collaboration/PROTOCOL.md) and [Roles & Responsibilities](collaboration/ROLES.md).

## Working principles
- The repository is the shared documentation source of truth, not proof that local application code has changed.
- Distinguish verified facts, hypotheses, proposals, and decisions.
- Link claims to reproducible evidence.
- Keep secrets, tokens, credentials, and personal data out of commits and review artifacts.
- Use a dedicated branch and pull request; do not merge without project-owner approval.
- Preserve unrelated work and platform documentation.
