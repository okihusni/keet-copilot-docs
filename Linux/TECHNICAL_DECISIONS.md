# Keet Copilot — Technical Decisions

_Last updated: 2026-11-14_

This document separates established decisions from proposals that still require review.

## Status vocabulary

- **Verified:** directly observed in source code or validated on the user's machine.
- **Approved:** accepted project direction.
- **Proposed — pending review:** design proposal not yet approved or implemented.
- **Open:** requires investigation or a decision.

## 1. Toolchain and repository

| Item | Status | Decision / evidence |
|---|---|---|
| Local source repo | Verified | `~/keet-copilot` |
| Active branch | Last known | `chore/electron45-vite8-migration` |
| Electron | Verified | 45.3.2 installed |
| electron-builder | Verified | 26.18.1 installed |
| electron-vite | Verified | 6.0.0 installed |
| Vite | Verified | 8.3.8 installed |
| @vitejs/plugin-react | Verified | 6.3.0 installed |
| TypeScript | Verified | strict mode |
| `npm run typecheck` | Previously passed | Passed after toolchain upgrade |
| `npm run build` | Previously passed | Passed after toolchain upgrade |

The migration changes in `package.json` and `package-lock.json` were uncommitted at the last recorded checkpoint. Preserve them. Do not reset, overwrite, or discard unrelated changes.

A safe dependency audit update was committed as `716bc0d chore: apply safe dependency security updates`. The migration branch was created from `main`.

## 2. Linux system audio capture

### 2.1 Capture source

**Approved direction:** use `parec` to capture the default sink's monitor source on Linux.

Command previously validated:

```bash
parec --device=alsa_output.pci-0000_05_00.6.analog-stereo.monitor \
  --raw --format=s16le --rate=48000 --channels=2
```

Implementation must dynamically discover the current default sink and monitor source using `pactl`; the example device name is not a permanent constant.

### 2.2 Volume and mute handling

**Approved requirements:**

1. Read and save the monitor source's exact per-channel volume values and mute state before modifying it.
2. Persist recovery information before changing the monitor.
3. Temporarily set the monitor source volume to 100% and unmute only if required.
4. Do not change the physical output sink volume.
5. Stop and confirm capture processes are gone before restoring the original monitor state.
6. Restore and read back the original volume and mute state.
7. Delete the recovery record only after verified restoration.
8. Never overwrite an unresolved recovery record.

The user's historical monitor volume was 13% and physical sink volume was 9%. Do not change these values without explicit permission.

### 2.3 Crash recovery

**Approved requirements:**

- Preserve a durable recovery record containing the original monitor source, per-channel volume values, and mute state.
- A failed or unverifiable restoration keeps the record and blocks new capture.
- If the original monitor source is unavailable, keep the record and explain the problem. Do not silently delete it.
- Process identity must be verified before sending signals. Never signal a process solely because a PID or process-group ID matches.
- Treat inaccessible process metadata conservatively; inability to inspect a process is not proof that no capture process remains.
- Do not restore the monitor while a capture process may still be active.

### 2.4 Lifeline-pipe helper

**Proposed — pending review; not approved or implemented.**

Claude's latest proposal uses a POSIX shell helper around `parec`:

- Electron spawns `/bin/sh` in a detached session/process group.
- The helper starts `parec`.
- A watcher blocks reading from a lifeline pipe whose writer is held by Electron.
- When Electron dies, the pipe closes; the watcher sees EOF and terminates `parec`.
- The helper's process group is persisted for recovery.
- Recovery scans `/proc` for processes in the expected group/session and checks command-line identity.
- The proposal supports older recovery records without `bootId` or process identity.

The design is promising but must not be treated as proven. The following issues remain open:

1. **Helper process identity:** persist and validate the helper's process start time in addition to boot ID and group ID. Verify process group, session, command line, and process start time before signalling.
2. **Unreadable `/proc` entries:** do not ignore `EACCES`/`EPERM` if doing so could cause recovery to falsely conclude that no capture process remains. Fail conservatively.
3. **Watcher lifecycle:** prove that the watcher subshell and its `cat` child cannot remain blocked or orphaned when `parec` exits unexpectedly, the helper receives SIGTERM, the helper receives SIGKILL, or Electron closes the lifeline.
4. **Descriptor inheritance:** verify that no process other than Electron retains a writer for the lifeline. The EOF guarantee depends on this.
5. **Helper independently killed:** the proposal acknowledges that independently SIGKILLing the helper may leave `parec` alive until recovery. Confirm that recovery can safely discover and terminate all relevant descendants.
6. **Crash during persistence:** initial record creation uses a temporary file, fsync, and hard link; update uses a temporary file and rename. Confirm temporary-file cleanup and failure behavior against the actual implementation.
7. **Power loss:** the proposal does not fsync the containing directory after record creation/rename. This limitation is documented and not yet addressed.

Do not apply the proposal until these concerns have been reviewed against the actual repository and shell behavior.

### 2.5 Legacy recovery records

**Approved requirement:** accept the previous record schema containing only `source`, `volumes`, and `muted`. If no reliable process identity exists, do not attempt to signal a process based on guessed identity. Restore saved audio state only when the source exists and read-back verification succeeds. Keep the record on failure.

## 3. Renderer audio and silence handling

**Approved requirements:**

- Integrate the main-process system audio stream into `AudioEngine` without breaking the microphone channel.
- Preserve speaker labels: `"you"` for microphone and `"them"` for interviewer/system audio.
- Keep the existing 16 kHz mono downstream STT contract.
- Add a wall-clock silence timer so utterance flushing does not depend solely on receiving another audio frame.
- Ensure renderer-side cleanup handles module-level HMR / React Fast Refresh; `did-navigate` alone is insufficient.

**Not yet implemented:** renderer Step 2, including `audio.ts`, `audioUtils.ts`, and HMR-safe stop handling.

## 4. AI and answer generation

- Suggestions must be grounded in the user's real CV, job description, experience, and technical notes.
- Never fabricate skills, work history, or domain experience.
- Prefer concise, natural spoken answers appropriate to approximately B1 English.
- Cloud AI is preferred by default over local model inference on the target hardware.

## 5. Development workflow decision

**Approved workflow:**

1. Keep existing project documentation as the shared source of truth.
2. Claude may research and author proposals.
3. Gemini/Antigravity inspects the actual local repository and implements only approved, bounded tasks.
4. ChatGPT independently reviews proposals, diffs, test output, and unresolved risks.
5. The project owner approves implementation and testing.
6. Update `PROJECT_STATUS.md`, `TECHNICAL_DECISIONS.md`, and `NEXT_TASK.md` after each meaningful milestone.

No implementation agent should commit changes without explicit approval. Do not let an agent reset or discard existing user changes.

## 6. References

- Upstream Keet: https://github.com/eeshsaxena/keet
- Shared project docs: https://github.com/okihusni/keet-copilot-docs
- Local source: `~/keet-copilot`
