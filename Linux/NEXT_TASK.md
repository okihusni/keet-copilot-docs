# Keet Copilot — Next Task

_Last updated: 2026-11-14_

## Current task

**Review the latest Claude Step 1 lifeline-pipe proposal against the actual local repository. Do not implement yet.**

Local repository: `~/keet-copilot`

Expected branch at the last checkpoint: `chore/electron45-vite8-migration`

## Inputs for the implementation agent

1. Read all four files in this documentation repository:
   - `PROJECT_CONTEXT.md`
   - `TECHNICAL_DECISIONS.md`
   - `PROJECT_STATUS.md`
   - `NEXT_TASK.md`
2. Read Claude's latest Step 1 proposal (the lifeline-pipe version supplied by the project owner).
3. Inspect the actual local repository and current Git working tree.

The latest Claude proposal must be supplied as an attachment or copied into the Antigravity conversation. It is not automatically available merely because this documentation references it.

## Instructions to Gemini / Antigravity

- Perform a read-only audit first.
- Confirm which documentation and proposal files were actually accessed.
- Inspect the actual source code; do not assume Claude's snippets match the current repository.
- Preserve valid decisions and existing uncommitted migration changes.
- Do not redesign the architecture without explaining why.
- Do not edit files, install packages, commit, reset, or discard changes during this audit.

### Review checklist

1. Is the helper command and shell script correct for Ubuntu's `/bin/sh`?
2. Does the lifeline pipe have exactly the intended writer ownership? Can any descendant inherit a writer and prevent EOF?
3. Does the watcher (`cat`) reliably exit when:
   - Electron closes the lifeline normally?
   - `parec` exits unexpectedly?
   - The helper receives SIGTERM?
   - The helper receives SIGKILL?
4. Can the watcher or `cat` remain orphaned or blocked?
5. Can `parec` survive if the helper is independently killed?
6. Is the process group identity strong enough? Require boot ID, helper PID/start time, process group, session, and command-line checks before signalling.
7. Are `/proc` permission errors handled conservatively? EACCES/EPERM must not be interpreted as proof that a process is absent.
8. Are process-group scans race-safe enough to avoid signalling unrelated processes after PID/PGID reuse?
9. Are recovery-file create/update/delete operations safe under interruption?
10. Are legacy records (`source`, `volumes`, `muted`) restored safely?
11. Is the monitor restored only after all capture processes are confirmed gone?
12. Does shutdown retain recovery information when cleanup is incomplete?
13. Are renderer HMR cleanup and idle-silence flush explicitly deferred to Step 2?

## Required audit output

Provide:

- Actual Git branch and `git status --short`.
- Exact files inspected.
- Summary of Claude's proposed design.
- Which parts are sound and should be retained.
- Specific defects or unverified assumptions, with file/line references.
- A minimal, incremental implementation plan.
- Commands/tests required after implementation, but do not run crash tests yet.

## Approval gate

Wait for explicit approval from the project owner before editing source files.

After approval, implement only the agreed Step 1 scope. Do not proceed to renderer Step 2 until:
- Step 1 typecheck passes.
- Relevant start/duplicate-start/stop/shutdown/reload tests pass.
- Crash recovery has been validated safely.
- The actual diff has been independently reviewed.

## Review handoff

After any implementation:
1. Do not commit.
2. Produce complete diffs, including new files.
3. List commands executed and actual outputs.
4. List modified/added/deleted files.
5. State known limitations and untested paths.
6. Wait for ChatGPT review and owner approval.
