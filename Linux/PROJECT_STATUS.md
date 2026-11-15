# Keet Copilot — Project Status

_Last updated: 2026-11-14_

## Overall status

**Project is in the Linux system-audio capture design/review stage.** The Electron/Vite migration has been validated. The Linux `parec` capture primitive has been tested. The main-process capture manager has not yet been implemented.

## Completed / verified

### Repository and toolchain

- [x] Cloned upstream Keet into `~/keet-copilot`.
- [x] Installed Node.js v22.23.4 and npm 10.9.9.
- [x] Created branch `chore/electron45-vite8-migration`.
- [x] Applied safe dependency security updates; commit: `716bc0d chore: apply safe dependency security updates`.
- [x] Upgraded Electron to 45.3.2.
- [x] Upgraded electron-builder to 26.18.1.
- [x] Upgraded electron-vite to 6.0.0.
- [x] Upgraded Vite to 8.3.8.
- [x] Upgraded @vitejs/plugin-react to 6.3.0.
- [x] `npm run typecheck` passed after migration.
- [x] `npm run build` passed after migration.
- [x] `npm run dev` launched the application and UI.

**Preserve:** at the last recorded checkpoint, `package.json` and `package-lock.json` contained uncommitted migration changes. Do not reset or discard them. Re-check `git status` before any new work.

### Linux audio investigation

- [x] Confirmed PipeWire with PulseAudio compatibility (`pactl` reports PulseAudio on PipeWire).
- [x] Confirmed `parec` is installed.
- [x] Identified default sink, microphone source, and monitor source during prior testing.
- [x] Successfully captured monitor-source audio with `parec`.
- [x] Confirmed captured signal is strong when monitor source is set to 100%.
- [x] Confirmed monitor-source volume controls capture independently of physical headphone output volume.
- [x] Restored historical test settings: monitor source 13%, physical sink 9%.

### Architecture investigation

- [x] Identified Electron's Linux loopback limitation for the existing display-capture approach.
- [x] Inspected the existing AudioEngine, pipeline, STT, IPC, preload, and shared API architecture.
- [x] Confirmed STT accepts 16 kHz mono Float32 audio and speaker identity is not needed by the STT implementation itself.
- [x] Identified that `ControlApp` owns the Pipeline and the overlay receives synchronized state.
- [x] Identified idle-silence behavior requiring a wall-clock flush timer.

## In progress

### Step 1 — Main-process Linux system audio capture manager

Claude authored multiple proposals. The latest proposal is the lifeline-pipe helper design. It is **not approved, not applied, and not typechecked**.

Current proposed scope:
- New `src/main/linuxSystemAudio.ts`.
- `pactl` source discovery and exact volume/mute read/restore.
- `parec` capture through a helper process.
- IPC start/stop/data/end channels.
- Shared API/types and preload bridge.
- Main-process shutdown and crash recovery.

Latest review concerns still open:
1. Persist helper process start time, in addition to boot ID and process group.
2. Validate process start time, process group, session and command line before signalling.
3. Treat `/proc` EACCES/EPERM as an inability to verify, not as proof a process is absent.
4. Verify watcher subshell and `cat` lifecycle for normal stop, unexpected `parec` exit, helper SIGTERM/SIGKILL, and Electron main-process SIGKILL.
5. Verify lifeline pipe writer inheritance: EOF must reliably mean Electron has exited.
6. Verify all descendants can be recovered if the helper is independently killed.
7. Review temporary recovery-file handling and failure paths.
8. Keep legacy recovery record support.
9. Do not restore audio while any capture process may still be alive.
10. Do not delete the recovery record unless original state is restored and verified.

No crash test should be run until the proposal has been reviewed and implemented.

## Not started

### Step 2 — Renderer integration and silence handling

- [ ] Integrate system audio IPC data into `AudioEngine`.
- [ ] Keep microphone and system audio channels independent.
- [ ] Add wall-clock silence flush handling.
- [ ] Add renderer-side cleanup for HMR / React Fast Refresh.
- [ ] Validate transcript speaker labels and pipeline behavior.

### Later milestones

- [ ] Verify end-to-end transcription from microphone and system audio.
- [ ] Verify interviewer-question detection and grounded answer generation.
- [ ] Validate UI/overlay behavior.
- [ ] Package and test the Linux application.
- [ ] Review security, privacy, consent and data-retention behavior.

## Latest known test notes

### `parec` capture

A monitor-source capture test produced a strong signal at 100% monitor volume. Physical sink volume changes did not materially change the monitor capture amplitude.

### Idle silence

A three-second `timeout` capture produced:

```text
Exit code: 124
196608 /tmp/keet-silence-test.raw
```

Expected continuous 3-second 48 kHz stereo s16le size: 576,000 bytes. The observed output was approximately 1.024 seconds. This suggests capture may stall during idle silence; the cause is not yet established.

## Current next action

Continue with the latest Claude lifeline-pipe proposal as the design baseline. Have Gemini/Antigravity inspect the actual repository and review the outstanding lifecycle concerns. ChatGPT independently reviews the findings. Do not modify source files until the owner approves the bounded implementation plan.
