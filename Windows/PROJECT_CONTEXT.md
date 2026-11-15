# Keet Copilot — Project Context
_Last updated: 2026-11-15_

## Purpose
Keet Copilot extends the open-source Keet desktop app into an AI interview-assistance workspace:
- Capture permitted interviewer/system audio.
- Transcribe speech near real time.
- Detect interviewer questions and follow-ups.
- Generate concise answer suggestions grounded in the user's actual CV, experience, job description, and technical notes.
- Present transcripts, suggestions, and controls.

Never fabricate professional experience. Use with appropriate consent and in accordance with interview rules. Do not build features to evade interviewer monitoring or bypass platform controls.

## User and product constraints
- QA Engineer with 9+ years overall: around 4 years game testing and 5+ years QA engineering.
- Around 3 years test automation; Playwright, TypeScript, Cypress, Postman, SQL; built a framework with 500+ automated tests.
- English approximately B1 progressing toward B2. Suggested spoken answers should be natural, concise, and easy to say.
- Prefer lightweight, low-cost services.
- Do not claim Web3, LLM evaluation, or other experience unless supported by the user's actual materials.
- Current interview workflow captures system/interviewer audio only. Microphone remains disabled unless explicitly changed by the owner.

## Workspace identity — important
Two workspace references exist and must not be conflated:
- Current active test workspace for Nov 15 Gemini Live work: `C:\Projects\keet` (Windows).
- Earlier Linux workspace in prior docs: `~/keet-copilot` (Ubuntu).

The active branch, Git status, commit, and relationship between these workspaces have not been independently reconfirmed. The reviewing agent MUST inspect the actual workspace and report repository root, branch, and `git status --short`. Do not assume the old Linux migration branch is active.

Upstream: https://github.com/eeshsaxena/keet  
Shared docs repo: https://github.com/okihusni/keet-copilot-docs

## Current Windows runtime verified
- Electron `33.4.11`; embedded Node `20.18.3`.
- `globalThis.WebSocket` was `undefined` in Electron main.
- `ws@^8.18.0` is a direct runtime dependency; `@types/ws@^8.18.0` is a dev dependency.
- `src/main/stt/geminiLive.ts` imports `WebSocket` from `ws`.
- electron-vite externalizes dependencies; development build emitted `require("ws")`.
- Packaging inclusion of `ws` has not been verified. Do not package/rebuild `Keet.exe` without owner approval.

Earlier Linux context remains historical/separate: Ubuntu GNOME X11, Ryzen 3 3250U, 10 GB RAM, PipeWire/PulseAudio, `pactl`/`parec`. Linux `parec`/lifeline-pipe proposals were not confirmed implemented and are not part of the current Windows Gemini Live task.

## Relevant architecture
Verify paths in the actual checkout:
- Main: `src/main/index.ts`, `src/main/ipc.ts`, `src/main/stt/index.ts`, `src/main/stt/geminiLive.ts`, `src/main/llm/index.ts`
- Preload: `src/preload/index.ts`
- Shared: `src/shared/types.ts`, `src/shared/ipc.ts`, `src/shared/gemini.ts`
- Renderer: `src/renderer/src/lib/audio.ts`, `audioUtils.ts`, `pipeline.ts`, `stt.ts`; control `ControlApp.tsx`, `LivePanel.tsx`, `SettingsPanel.tsx`; overlay `OverlayApp.tsx`; state `store.ts`.

## Current audio/STT constraints
- Capture interviewer/system audio only; microphone disabled.
- Gemini Live audio: PCM16, 16 kHz, mono (verified in Test B).
- Interviewer speaker label: `"them"`.
- DEV uses Gemini Live STT.
- LIVE uses Groq batch STT. Preserve provider separation.
- Do not expose API keys.
- Preserve `contentProtection: true`.
- Do not change VAD/audio format as part of the transcript-finalization task.

## Roles
- Antigravity orchestrator: inspect and implement only approved bounded tasks.
- Antigravity Sonnet: independent second-pass reviewer; read-only unless explicitly authorized to edit.
- ChatGPT: independent technical reviewer and QA coordinator.
- Owner: approves changes, real API tests, packaging, and commits.

No agent may commit, reset, clean, overwrite unrelated changes, or discard user work without explicit approval.
