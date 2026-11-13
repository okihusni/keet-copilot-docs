# Keet Copilot — Project Context

_Last updated: 2026-11-14_

## 1. Project purpose

Keet Copilot is a personal AI Interview Copilot built by extending the open-source Keet desktop application.

The goal is to provide an interview-assistance workspace that can:

- Capture permitted meeting audio, with microphone and system audio treated as separate channels.
- Transcribe speech in near real time.
- Identify interviewer questions and follow-up questions.
- Generate concise, natural answer suggestions grounded in the user's actual CV, experience, job description, and technical notes.
- Maintain a local knowledge base and interview history.
- Present transcripts, answer suggestions, talking points, and controls in a desktop UI.

The application must not fabricate professional experience or imply experience the user does not have. Use only with appropriate consent and in accordance with interview rules. Do not build features intended to evade interviewer monitoring or bypass platform controls.

## 2. User and response constraints

- Primary environment: Linux desktop, Ubuntu GNOME on X11.
- Prefer cloud AI services over local Whisper/LLM inference by default because the target machine is modest.
- Keep the application lightweight.
- The user's English level is approximately B1 and moving toward B2. Suggested spoken answers should sound natural and use manageable sentence structures.
- Target additional AI-service spending: ideally $0, with approximately $10/month as an upper preference if the value is clear.

## 3. User's professional background for answer grounding

- Approximately 9+ years in QA: around 4 years in game testing and 5+ years as a QA Engineer.
- Approximately 3 years of test automation experience.
- Tools include Playwright, TypeScript, Cypress, Postman, and SQL.
- Built an automation framework with 500+ automated tests.
- Interested in SDET and application security.
- Do not claim Web3, LLM evaluation, or other experience unless supported by the user's actual materials.

## 4. Upstream and local repositories

- Upstream Keet: https://github.com/eeshsaxena/keet
- Local source repository: `~/keet-copilot`
- Shared documentation repository: https://github.com/okihusni/keet-copilot-docs

The documentation repository is intended to be the shared source of truth for ChatGPT, Gemini/Antigravity, Claude, and the project owner. It must distinguish verified facts, approved decisions, proposals, and unresolved questions.

## 5. Technology and local environment

- Electron desktop application with React renderer.
- Electron Vite build tooling.
- TypeScript strict mode.
- Node.js v22.23.4; npm 10.9.9.
- Ubuntu GNOME X11.
- AMD Ryzen 3 3250U, 10 GB RAM, Radeon Vega integrated GPU, 4 GB swap.
- PipeWire with PulseAudio compatibility.
- `pactl` and `parec` are available.
- Wired headset connected through a 3.5 mm audio jack.

Known PulseAudio-compatible devices from prior inspection:

- Default sink: `alsa_output.pci-0000_05_00.6.analog-stereo`
- Default source: `alsa_input.pci-0000_05_00.6.analog-stereo`
- Monitor source: `alsa_output.pci-0000_05_00.6.analog-stereo.monitor`

These device names can change. Discover the current default sink and corresponding monitor dynamically; do not hard-code the above names.

## 6. Existing application architecture

- `src/main/index.ts`: Electron main-process bootstrap, windows, permissions, shortcuts, and lifecycle.
- `src/main/ipc.ts`: IPC handlers.
- `src/preload/index.ts`: context bridge and renderer-facing API.
- `src/shared/`: shared IPC channel names, API contracts, and types.
- `src/renderer/src/lib/audio.ts`: renderer-side audio capture/processing and VAD.
- `src/renderer/src/lib/audioUtils.ts`: PCM helpers including resampling and WAV encoding.
- `src/renderer/src/lib/pipeline.ts`: transcription and answer-generation pipeline.
- `src/renderer/src/lib/stt.ts`: local/cloud STT integration.
- `ControlApp.tsx`: owns the Pipeline; overlay receives synchronized state.

Only the control window currently owns and runs the Pipeline.

## 7. Audio architecture facts

The existing `AudioEngine`:

- Captures microphone audio through `getUserMedia`.
- Attempts system audio through `getDisplayMedia({ video: true, audio: true })`.
- Uses an AudioContext, MediaStreamSource, ScriptProcessorNode, and zero-gain output node.
- Performs VAD using RMS, voice buffers, silence gap, and a maximum utterance duration.
- Resamples utterances to 16 kHz before passing them to the pipeline.
- Labels channels as `"you"` (microphone) and `"them"` (system/interviewer).

Electron's `audio: "loopback"` display-capture path is not supported on Linux in the same way as Windows and macOS. The existing system-audio capture path therefore needs a Linux-specific implementation.

The chosen Linux capture primitive is `parec` reading the default sink's `.monitor` source. The renderer integration is still pending.

## 8. Validated audio observations

The following were previously tested on the user's machine:

- `parec` can capture audio from the PulseAudio-compatible monitor source.
- YouTube playback was used as a proxy for system audio.
- At 100% monitor-source volume, captured PCM had a strong measurable signal (previous observation: peak 23363, RMS approximately 4772).
- Monitor-source volume affects capture independently of the physical headphone sink volume.
- The original monitor-source volume was 13%; physical output sink volume was 9%.

**Do not change the user's current audio levels as part of development without explicit permission.** The previously restored values were monitor source 13% and sink 9%; treat these as historical observations, not guaranteed current values.

A three-second idle-silence experiment returned 196,608 bytes rather than the expected 576,000 bytes for continuous 48 kHz stereo s16le. This suggested a bounded burst followed by a stall while idle; the exact cause was not established. The renderer design therefore needs a wall-clock silence timer independent of incoming PCM frames.

## 9. Product and implementation constraints

- Keep microphone and system audio channels distinguishable.
- Keep capture start/stop ownership explicit.
- Preserve and restore the exact original monitor-source per-channel volume and mute state.
- Do not alter physical headphone output volume.
- Do not restore audio settings while the capture process may still be alive.
- Do not overwrite an unresolved recovery record.
- Do not claim a process has stopped without evidence.
- A missing original monitor source must not silently cause the recovery record to be deleted.
- Handle renderer teardown and HMR separately; a full navigation event does not cover module-level HMR or React Fast Refresh.
- Do not begin renderer Step 2 until the main-process capture manager is reviewed, implemented, and validated.

## 10. Working roles

- **Claude:** technical research and implementation-proposal author. Prior Claude proposals remain valuable design input, but are not automatically approved or implemented.
- **Gemini / Antigravity:** local repository inspection, approved implementation, and test execution.
- **ChatGPT:** independent technical reviewer, QA coordinator, and project coordinator.
- **Project owner:** approves changes, runs/observes relevant tests, and decides when to proceed.

No AI should silently discard previous decisions or redesign the system without explaining why.
