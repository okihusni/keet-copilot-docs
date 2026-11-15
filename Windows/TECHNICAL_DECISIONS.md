# Keet Copilot — Technical Decisions
_Last updated: 2026-11-15_

Status vocabulary: **Verified** = observed in source/tests/runtime; **Approved** = accepted constraint; **Proposed — pending review** = not yet approved/verified; **Open** = needs evidence.

## 1. Gemini Live runtime compatibility
| Item | Status | Evidence |
|---|---|---|
| Active test workspace | Verified for Nov 15 | `C:\Projects\keet` |
| Electron / Node | Verified | `33.4.11` / `20.18.3` |
| Main-process `globalThis.WebSocket` | Verified unavailable | `typeof globalThis.WebSocket === "undefined"` |
| Node WebSocket | Implemented and mocked-tested | `ws@^8.18.0` runtime dependency |
| `@types/ws` | Implemented | `^8.18.0` dev dependency |
| `binaryType` | Verified in Test A | `"arraybuffer"` |
| Gemini model | Verified configured | `models/gemini-3.5-transcribe-live` |
| Packaged inclusion of `ws` | Open | No packaging test; do not package without approval |

`defaultWebSocketFactory()` in `src/main/stt/geminiLive.ts` uses `new WebSocket(url)` from `ws`. Preserve the factory abstraction and session lifecycle.

## 2. Test A — real Gemini Live connection: PASS (Nov 15)
- Real Electron DEV app connected to Gemini Live.
- WSS opened; `setupComplete` arrived as a 26-byte ArrayBuffer and decoded.
- Manager returned `{ ok: true, sessionId: 1 }` and emitted connected status.
- WebSocket open: 219 ms; `setupComplete`: 474 ms; renderer start IPC round-trip: 584 ms.
- Stop closed socket with code 1000; cleanup succeeded.
- No sanitized WebSocket errors.
- DEV key was not exposed; `contentProtection: true`.
- No source/config changes during the real Test A run.

## 3. WebSocket decoding/order
Implemented and mocked-tested:
- `ws.binaryType = "arraybuffer"`.
- String, ArrayBuffer, and ArrayBufferView decoding.
- Blob fallback with serialized async message processing to preserve FIFO ordering.
- Session/generation and active-socket guards.
Tests cover ArrayBuffer setup, Blob fallback, mixed Blob then ArrayBuffer ordering, and Node `ws` loopback. Preserve these guarantees.

## 4. Test B — real audio/transcription: PARTIAL PASS (Nov 15)
Verified:
- System audio capture worked; microphone disabled.
- Interviewer-style TTS question played through Windows system audio.
- PCM16 16 kHz mono streamed: 81 frames, 221,184 bytes, about 6.912 seconds.
- 12 interim transcription updates arrived.
- One `serverContent.inputTranscription.text` arrived 363 ms after local `activityEnd`.
- No `serverContent.turnComplete` observed.
- Manager waited about 10 seconds and discarded the received text because it required `turnComplete`.
- No final transcript reached UI.
- LLM start was intercepted (0 invocations); question detection/answer generation were not tested.
- Session cleanup succeeded; no source/config changes during Test B.

Expected: `Oki, could you tell me about your experience as a QA Engineer and the automation tools you've used?`
Interim: `Ok, could you tell me about your experience as a QA engineer and the automation tools you've used?`
Raw `inputTranscription.text`: `Could you tell me about your experience as a QA engineer and the automation tools you've used?`
Leading `Oki` was misrecognized/omitted. Track separately after finalization is fixed.

## 5. Transcript finalization — proposed, not yet implemented/verified
Observed root cause: manager stores `inputTranscription.text` but commits only when `serverContent.turnComplete === true`. The transcription-only model did not emit `turnComplete` in Test B, so the 10-second timeout discarded the text.

Critical evidence gap: Test B hook recorded only `inputTranscription.text`, not the full raw object. The `inputTranscription.finished` value is **unknown**. Do not assume true, false, or omitted.

Proposed direction:
- Treat `inputTranscription` as the authoritative transcription result for this transcription-only model, not `turnComplete`.
- Keep `interimInputTranscription` provisional; never commit interim text as final.
- Associate transcription with the correct local VAD utterance.
- If transcription arrives while local audio is active, hold until local `activityEnd` to avoid triggering downstream auto-answer mid-sentence.
- Once local audio has ended and authoritative `inputTranscription` is available, commit exactly once.
- Preserve ordering and multi-segment behavior. Do not assume one message per local utterance unless supported by protocol/runtime evidence.
- Do not add arbitrary 250 ms debounce without evidence.
- Preserve bounded timeout for genuinely missing final transcription.
- Late `turnComplete` must not affect a newer active utterance.
- Make commit idempotent; clear timers on commit, timeout, stop, invalidation, and cleanup.

Open design issue: Live Transcription docs describe `inputTranscription` as finalized and independent of other server messages; ordering relative to other server messages is not guaranteed. Sonnet must challenge the event-to-local-turn association and multi-segment algorithm. Do not infer that `finished` is a required/reliable flag for this model without evidence.

## 6. Finalization invariants
1. Interim text is never final.
2. Each local utterance emits at most one final event.
3. Final text is not discarded merely because `turnComplete` is absent.
4. Segment order is preserved where messages belong to one utterance.
5. Pre-`activityEnd` transcription is held.
6. Stale session/generation events cannot affect a new session.
7. Late `turnComplete` cannot complete a newer active utterance.
8. Timers clear on every completion/invalidation/stop path.
9. Missing authoritative final transcription yields a bounded observable error; do not promote interim text silently.
10. DEV Gemini Live / LIVE Groq routing remains unchanged.

## 7. Workflow constraints
- Antigravity orchestrator implements only approved bounded tasks.
- Antigravity Sonnet independently reviews the actual diff; review-only means no edits.
- ChatGPT reviews diffs, tests, and runtime evidence.
- Owner approval required before real API retests, packaging, commits, or scope changes.
- Never expose API keys; preserve `contentProtection`.
- Do not rebuild/package `Keet.exe` without explicit approval.
- Do not modify unrelated files or discard user changes.
- On Windows PowerShell, use `npm.cmd` for npm scripts if needed.

## 8. Historical Linux workstream
Prior docs cover Linux PipeWire/PulseAudio capture via `parec`, recovery records, and a lifeline-pipe helper. That is a separate historical/parallel workstream. Lifeline-pipe was previously pending review, not approved/implemented. Do not confuse it with current Windows Gemini Live work.
