# Keet Copilot — Next Task
_Last updated: 2026-11-15_

## Current task
**Use Antigravity Sonnet for an independent, read-only review of the orchestrator's Gemini Live transcription-finalization implementation.**

This snapshot does not confirm that implementation is complete. First inspect the actual workspace and report current state.

Active test workspace: `C:\Projects\keet` (Windows). Older docs refer to `~/keet-copilot` (Ubuntu); do not assume same checkout/branch.

## Read first
Read all four docs:
1. `PROJECT_CONTEXT.md`
2. `TECHNICAL_DECISIONS.md`
3. `PROJECT_STATUS.md`
4. `NEXT_TASK.md`

Then inspect the actual repo and the orchestrator's full implementation report/diff. Do not rely only on summary text.

## Background
Test A passed against real Gemini Live in Electron DEV:
- Electron 33.4.11 / Node 20.18.3.
- Node `ws` opened WSS; `setupComplete` decoded from ArrayBuffer.
- Manager connected; start IPC round-trip 584 ms; clean close code 1000.

Test B partial pass:
- System audio capture and PCM16 16 kHz mono transmission worked.
- 12 interim transcription updates.
- One `serverContent.inputTranscription.text` arrived 363 ms after local `activityEnd`.
- No `serverContent.turnComplete` observed.
- Manager waited ~10 seconds and discarded the transcript.
- The Test B hook recorded only `.text`; `inputTranscription.finished` is unknown.

Proposed fix: treat `inputTranscription` as the primary finalization signal for `models/gemini-3.5-transcribe-live`, respect local `activityEnd`, and preserve ordering, multi-segment behavior, idempotency, and session guards.

## Sonnet review checklist
1. Confirm actual repo root, branch, `git status --short`, Electron/Node and dependency versions.
2. List exact docs/source files read.
3. Inspect complete diff, including untracked files.
4. Verify no `turnComplete` requirement for transcription-only finalization.
5. Verify interim text never becomes final.
6. Challenge association of `inputTranscription` messages to local VAD utterances.
7. Challenge multiple-segment behavior and any debounce; require protocol/runtime evidence rather than assumptions.
8. Verify transcription received before `activityEnd` is held.
9. Verify late `turnComplete` cannot affect a newer active utterance.
10. Verify duplicate finalization is impossible/idempotent.
11. Verify timers clear on commit, timeout, stop, invalidation, reconnect, and cleanup.
12. Verify session/generation guards reject stale events.
13. Verify missing authoritative final transcription still yields a bounded error.
14. Verify tests are meaningful and would fail against the old implementation.
15. Check DEV Gemini Live / LIVE Groq separation, mic disabled, API-key confidentiality, and `contentProtection`.
16. Explain unrelated file/dependency changes.

## Strict review-only constraints
- Do not edit source/tests.
- Do not install/update dependencies.
- Do not commit, reset, clean, or discard changes.
- Do not package/rebuild `Keet.exe`.
- Do not run a real Gemini API test.
- Do not expose API keys.
- Do not change VAD, audio capture, audio format, model, endpoint, or provider routing.
- Do not broaden scope.

If implementation is incomplete or flawed, report findings and wait. Do not fix during this pass.

## Required output
- Actual repo root, branch, `git status --short`.
- Exact files inspected.
- Implementation summary.
- Findings: Blocker / High / Medium / Low with file/line references.
- Test coverage assessment.
- Unresolved protocol assumptions/edge cases.
- Verdict: Approve for next review / Request changes / Insufficient evidence.
- Explicit confirmation: no files modified and no real API test run.

## Approval gate
Return Sonnet's complete report and orchestrator's full diff for independent review. Only after owner/ChatGPT approval may findings be fixed or a new real Test B be run. Do not proceed to Tests C–E until final transcript commitment is verified.

Expected implementation files:
- `src/main/stt/geminiLive.ts`
- `tests/gemini-live-stt.test.ts`

Any additional changed file must be explained and justified. No packaging or `Keet.exe` rebuild is authorized.

---

## Queued implementation backlog

### Q-01 — AI E2E Performance, Token Usage & Cost Tracking
**Status:** Queued / not authorized for implementation  
**Priority:** After the current Gemini Live STT finalization workstream and its approval gate  
**Target:** Windows Electron app (`Keet.exe`), with provider-neutral instrumentation where practical

**Objective:** Capture reliable per-request end-to-end latency, model token usage, and estimated API cost, then make the data available for local diagnostics and performance reporting.

**Required metrics**
- End-to-end response time: start when the user submits/triggers a request; stop when the complete answer is rendered and available in the UI.
- Time to first token (TTFT), when streaming is supported.
- Input, output, and total token counts, using provider-reported usage metadata whenever available.
- Estimated per-request API cost, calculated from the actual provider/model and a traceable pricing configuration.
- Provider, model, request/correlation ID, timestamps, completion status, and error/cancellation outcome.

**Reporting**
- Aggregate average, median, P95, and maximum E2E response time.
- Aggregate token consumption and estimated cost by provider/model and over a selected period.
- Keep STT/transcription duration and LLM generation duration separate when both can be measured reliably; do not confuse either with the full E2E measurement.

**Implementation constraints**
- Prefer local structured logging/storage (JSONL or SQLite); do not introduce a paid monitoring service.
- Use a monotonic timer for elapsed durations.
- Treat unavailable token usage or pricing as unknown/null; never fabricate counts or costs.
- Label cost as an estimate and retain the pricing source/version or effective date used for calculation.
- Avoid logging interview transcript content or unnecessary personal data.
- Add offline unit tests with mocked provider responses and usage metadata. No live/paid API calls without explicit owner approval.
- Do not package/rebuild `Keet.exe` until separately approved.

**Acceptance criteria**
1. E2E timing measures the complete user-visible request lifecycle, including rendering completion.
2. TTFT is recorded where streaming supports it; otherwise it is explicitly unavailable.
3. Token counts use provider usage metadata, with missing values represented as unknown.
4. Cost calculations are reproducible from provider, model, token categories, and pricing configuration.
5. Logs correlate timing, usage, and cost for the same request.
6. Aggregates include average, median, P95, and maximum, with clear sample counts.
7. Offline tests cover success, failure, cancellation, missing usage metadata, and pricing/model changes.
8. No additional paid services or live API calls are introduced without approval.

**Approval gate:** This is a backlog item only. Do not begin implementation until the current STT workstream is complete and the owner explicitly authorizes Q-01.


---

## Future product goals

### F-01 — Mock Interview with Continuous Knowledge Improvement
**Status:** Future product goal / not scheduled / not authorized for implementation  
**Priority:** After the current core Copilot workstreams are stable; sequence to be decided by the owner  
**Scope:** A future Mock Interview mode that helps the user practice interviews and, with review safeguards, gradually improves the shared knowledge library used by Copilot.

**Objective:** Let users practice realistic interviews and turn useful, genuinely new learning outcomes into candidate knowledge entries. Once reviewed and approved, those entries become available to Copilot during future interviews.

**Proposed workflow**
1. User starts a mock interview, optionally using a job description and selected role/topic.
2. Keet asks questions and records the user's answers and session feedback.
3. Keet identifies potential knowledge additions, corrections, strengths, and learning gaps from the session.
4. Keet creates candidate knowledge entries with traceable links to the mock interview session and relevant source material.
5. User can review, edit, approve, reject, or merge each candidate entry.
6. Only approved entries are added to the shared knowledge library and made available to Copilot retrieval.
7. Future mock interviews and Copilot sessions can use the updated knowledge.

**Knowledge integrity and safety requirements**
- Reuse the existing five-category knowledge model: verified professional experience, verified practical skills, supported technical methodology, technical study and learning, and explicit knowledge gaps.
- Never automatically promote an AI-generated suggestion, hypothetical answer, or unverified claim into verified professional experience or verified practical skills.
- Distinguish what the user has actually done, what the user understands, what the user is currently learning, and what remains a knowledge gap.
- Keep source provenance for every candidate and approved knowledge entry, including the originating session and user approval.
- Detect duplicates and conflicting entries; provide review/merge behavior rather than silently overwriting existing knowledge.
- Preserve revision history and allow approved knowledge to be corrected or withdrawn.
- Mock interview feedback may identify learning needs, but feedback alone is not proof of new professional experience.

**Cost and architecture constraints**
- Local-first design; no dependency on paid external APIs.
- Prefer deterministic rules, question banks, templates, and existing local processing where practical.
- Any optional local model must be evaluated against the user's hardware and remain optional.
- Reuse the existing knowledge library and retrieval architecture rather than creating a separate Copilot knowledge store.
- Keep Mock Interview and Copilot as separate modes with a shared knowledge layer.

**Success criteria**
1. Users can complete mock interview sessions and review their questions, answers, and feedback.
2. Keet can propose useful candidate knowledge entries from session outcomes.
3. Candidate entries are categorized using the existing knowledge taxonomy and retain source provenance.
4. Nothing is added to verified experience or practical skills without explicit user approval and appropriate evidence.
5. Approved entries become retrievable by Copilot; rejected or unreviewed entries do not.
6. Duplicate/conflicting knowledge can be identified and reviewed without silent data loss.
7. The core workflow can operate without paid API calls.

**Approval gate:** This is a future product goal only. It is not part of the current implementation scope, does not change the active STT or Stage 3.1B workstreams, and is not authorization to begin design implementation, source changes, model integration, or paid API usage.
