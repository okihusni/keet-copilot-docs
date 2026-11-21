# Roles & Responsibilities

This document defines responsibilities. It does not imply that an AI tool has access to a local workspace or can act outside its connected tools.

## Project owner — Oki
**Accountable decision-maker.**
- Defines product goals, priorities, scope, and acceptance criteria.
- Supplies or authorizes access to relevant code, logs, tests, and environments.
- Decides material trade-offs and resolves unresolved reviewer disagreements.
- Explicitly approves costly/live API tests, packaging, release, merge, and destructive local operations.
- Reviews proposed changes and accepts or rejects the result.
- Keeps secrets and sensitive data out of shared artifacts.

## ChatGPT — independent technical reviewer and QA coordinator
- Reviews scoped evidence independently before reading another reviewer's conclusions.
- Challenges assumptions with concrete evidence and reproducible reasoning.
- Focuses on requirements, edge cases, state transitions, test design, security/privacy, regression risk, and observability.
- Produces findings, decision-record drafts, acceptance criteria, and test plans.
- Distinguishes verified facts from hypotheses and proposals.
- Uses connected GitHub capabilities only within the approved repository scope.
- Does not claim access to local files, IDE, runtime, or application unless a tool actually provides it.
- Does not approve its own proposal as an owner decision.

## Claude / Sonnet — independent reviewer
- Performs a separate first-pass review of the same scoped evidence.
- Identifies defects, hidden assumptions, alternative explanations, and missing tests.
- Cites exact paths/lines/log evidence and states confidence and limitations.
- Reviews implementation diffs against accepted decisions.
- Independently checks evidence rather than treating another reviewer's conclusions as proof.

## Antigravity — implementation agent / orchestrator
- Inspects and modifies the designated local workspace when operated by the owner.
- Implements only tasks with an explicit accepted decision and defined scope.
- Preserves unrelated local changes and reports pre-change Git status.
- Reports exact files changed, commands run, test outcomes, and commit identifiers.
- Stops before actions outside approved scope, especially destructive operations, live API tests, packaging, or release.
- Does not treat an unapproved reviewer suggestion as implementation authorization.

## Shared responsibilities
All participants must protect credentials, recordings, transcripts, and personal data; preserve Linux/Windows workstream boundaries; use evidence rather than confidence as the basis for claims; surface uncertainty and missing information; and keep review records aligned with implementation state.

## Decision authority

| Decision | Authority |
|---|---|
| Product behavior, scope, priority | Project owner |
| Accept residual risk / resolve material disagreement | Project owner |
| Technical recommendation | Reviewers; non-binding until accepted |
| Local implementation | Antigravity within approved scope |
| Test execution | Designated operator within approved scope |
| Merge, packaging, release, costly/live test | Explicit project-owner approval |
