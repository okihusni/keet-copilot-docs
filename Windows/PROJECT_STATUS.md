# Keet Copilot — Project Status
_Last updated: 2026-11-15_

## Overall status
Active workstream: Windows Electron DEV-mode Gemini Live STT. Real connection, system-audio capture, and interim transcription work. Current blocker: final transcript commitment. The manager waits for `serverContent.turnComplete`, which was not observed from `gemini-3.5-transcribe-live` in Test B.

A finalization implementation prompt has been prepared. This snapshot does **not** confirm that the orchestrator has implemented the fix. Next: orchestrator implementation, then independent Antigravity Sonnet review.

## 1. Current workspace/runtime
- Active test workspace: `C:\Projects\keet` (Windows).
- Electron `33.4.11`; embedded Node `20.18.3`.
- Main-process `globalThis.WebSocket` is undefined.
- `ws@^8.18.0` runtime dependency; `@types/ws@^8.18.0` dev dependency.
- Recheck actual branch and `git status --short`; older docs refer to `~/keet-copilot` and a Linux migration branch.
- `Keet.exe` has not been rebuilt/packaged for this workstream.

## 2. Completed/verified
### WebSocket compatibility
- Main process switched from `globalThis.WebSocket` to Node-compatible `ws`.
- `WebSocketLike` factory abstraction retained.
- ArrayBuffer decoding and Blob fallback/FIFO retained.
- Added local `WebSocketServer` loopback regression test.
- Mocked tests passed: audio isolation 16/16; tray/overlay 7/7; environment mode 18/18; Gemini Live 12/12.
- `npm.cmd run typecheck`, `npm.cmd run lint`, `npm.cmd run build`, and `git diff --check` passed.
- No `Keet.exe` packaging.

### Test A — real Gemini Live: PASS
- WSS opened in real Electron DEV app.
- `setupComplete` received and decoded.
- Manager connected; start IPC round-trip 584 ms.
- Socket closed cleanly (1000); cleanup succeeded.
- No key exposure; `contentProtection` true.

### Test B — real system audio/transcription: PARTIAL PASS
- System audio captured; mic disabled.
- PCM16 16 kHz mono transmitted: 81 frames / 221,184 bytes / ~6.912 seconds.
- 12 interim updates received.
- One `inputTranscription.text` arrived 363 ms after local `activityEnd`.
- No `turnComplete` observed.
- Manager timed out after ~10 seconds and discarded transcript; UI final list empty.
- LLM start intercepted (0 calls); question detection and answer generation not tested.
- Cleanup succeeded; source/config unchanged during test.

Transcript:
- Expected: `Oki, could you tell me about your experience as a QA Engineer and the automation tools you've used?`
- Interim: `Ok, could you tell me about your experience as a QA engineer and the automation tools you've used?`
- Raw final frame text: `Could you tell me about your experience as a QA engineer and the automation tools you've used?`
- Leading `Oki` lost/misrecognized; separate issue after finalization.

## 3. Blocker and proposed fix
Root cause: `inputTranscription.text` is stored, but only `turnComplete` triggers commit. No `turnComplete` arrived, so timeout discarded the text.

The Test B instrumentation did not capture the complete raw `inputTranscription` object; `finished` is unknown.

Proposed behavior:
- `inputTranscription` is primary authoritative finalization signal for the transcription-only model.
- Interim remains provisional.
- Respect local `activityEnd` before downstream commit.
- Preserve message ordering and multi-segment handling.
- No arbitrary debounce without evidence.
- Prevent duplicate finals, late-event cross-turn bleed, and stale-session effects.
- Retain bounded error timeout for missing authoritative final transcription.

## 4. Next task: orchestrator implementation, then Sonnet review
Expected implementation scope:
- Production: `src/main/stt/geminiLive.ts`
- Tests: `tests/gemini-live-stt.test.ts`

Required test cases:
1. `inputTranscription` after `activityEnd` commits without `turnComplete`.
2. `finished: true` works if supported.
3. Omitted `finished` works.
4. Interim-only text is never final.
5. Multiple segments preserve order and yield one final event where protocol semantics support it.
6. Transcription before local `activityEnd` is held.
7. Late `turnComplete` cannot corrupt next active utterance.
8. Both signals do not duplicate final.
9. Missing final transcription times out with an error.
10. Stop/session invalidation blocks late commits.

No real API test during implementation. Run relevant/full mocked suites, typecheck, lint, build, and `git diff --check`. Do not package Keet.exe.

Sonnet review must be read-only:
- Inspect actual repo, branch, status, complete diff, source, and tests.
- Challenge event association, multi-segment behavior, activity boundaries, late `turnComplete`, duplicate protection, timer cleanup, session guards, and stop/reconnect behavior.
- Check tests would fail against old code and truly cover behavior.
- Report findings with file/line references and severity.
- Do not edit/install/commit/reset/clean/package or run real API tests.
- Wait for owner approval.

## 5. Not yet tested
- Real API final transcript commitment after fix.
- Actual `inputTranscription.finished` field.
- Leading-word preservation (`Oki`).
- Rapid utterances, network recovery, session rotation.
- Question detection and LLM answer generation.
- Packaged app and `ws` inclusion.
- Packaged `Keet.exe`.

## 6. Historical Linux workstream
Linux `parec`/PipeWire capture and lifeline-pipe proposal remain separate historical/parallel work. Prior status: lifeline-pipe pending review, not implemented. Do not confuse with current Windows Gemini Live work.
