# Keet Interview Copilot — Technical Decisions

Last updated: 2026-11-13

This document records approved decisions and unresolved questions.
Proposals are not approved decisions until explicitly accepted.

## 1. Approved Decisions

| ID | Topic | Decision |
|---|---|---|
| TD-001 | Foundation | Evaluate upstream Keet before considering a rewrite. |
| TD-002 | Source changes | Do not modify source before baseline validation. |
| TD-003 | Runtime | Follow repository Node requirement: >=20. |
| TD-004 | Dependencies | Use npm ci because package-lock.json exists. |
| TD-005 | Inference | Prefer cloud inference on current hardware. |
| TD-006 | Audio | Inspect PipeWire topology before changing audio routing. |
| TD-007 | Rollback | Audio changes must be reversible and documented. |
| TD-008 | Screen sharing | Do not rely on hiding the app from screen capture. |
| TD-009 | Answer grounding | Answers must be based on verified user information. |
| TD-010 | API security | Never expose API keys in renderer code or Git. |

## 2. Verified Repository Facts

From the checked-out package.json:

- Package: keet
- Version: 0.1.0
- Node requirement: >=20
- Electron: ^33.2.1
- electron-vite: ^2.3.0
- React: ^18.3.1
- TypeScript: ^5.7.2

Available scripts:

- npm run dev
- npm run build
- npm run preview
- npm run typecheck
- npm run lint
- npm run package
- npm run package:linux

The package-lock.json file exists.

Note: declared dependency ranges do not necessarily represent exact installed versions. Inspect the lockfile or npm ls when exact versions matter.

## 3. Architecture Options — Not Yet Finalized

### Audio capture

- Inspect the existing implementation first.
- Test system audio and microphone independently.
- Confirm that meeting audio remains audible while capture is active.
- Understand the existing PipeWire topology before creating virtual sinks.
- Test device switching, suspend/resume, echo, and feedback.

### Speech-to-text

Compare:
1. A genuine streaming/live transcription API.
2. A cloud batch/chunked transcription API as fallback.

Verify model identifiers, regional availability, audio formats, session limits, reconnect behavior, latency, quotas, pricing, and data terms.

Do not assume an AI-suggested model ID is real or currently available without checking official documentation.

### Answer generation

- Use a separate provider abstraction.
- Generate concise, natural answers.
- Ground suggestions in retrieved user information.
- Return uncertainty or missing evidence instead of fabricating facts.
- Consider structured output and streaming UI updates.

### Knowledge retrieval

- Start with local document ingestion.
- Retrieve only relevant excerpts.
- Keep original personal documents local.
- Maintain evidence references for answer verification.

### Security

- Keep API keys in the Electron main process or OS-backed secure storage.
- Validate IPC payloads.
- Restrict exposed preload APIs.
- Avoid logging sensitive transcripts or credentials.
- Review dependency vulnerabilities before upgrading packages.

## 4. Open Questions

1. Which Keet files implement audio capture, VAD, transcription, and answer generation?
2. Does the original app run on Ubuntu GNOME X11?
3. Does the current implementation capture system audio, microphone audio, or both?
4. Which cloud STT provider best balances accuracy, latency, privacy, and cost?
5. Which answer-generation model is appropriate?
6. What are the measured CPU, RAM, and end-to-end latency baselines?
7. What consent and data-retention controls should be implemented?

## 5. Change Control

Before implementation, every proposed architectural change must include:

1. Problem and supporting evidence.
2. Alternatives and trade-offs.
3. Security, privacy, cost, and maintenance implications.
4. Test plan and rollback strategy.
5. Explicit user approval.
