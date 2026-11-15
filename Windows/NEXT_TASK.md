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
