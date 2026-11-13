# Keet Interview Copilot — Next Task

Last updated: 2026-11-13

## Active Task: Complete the First Build Baseline

Objective: validate the original Keet project after dependency installation without modifying source code or audio configuration.

### Step 1 — Inspect dependency audit findings

Run:

npm audit

Review:
- Which packages are affected?
- Are findings direct or transitive?
- Are they production or development dependencies?
- Are non-breaking fixes available?
- Does the critical finding affect runtime code or development tooling?

Do not run npm audit fix or npm audit fix --force yet.

### Step 2 — Run TypeScript checks

Run:

npm run typecheck

Capture the full output if it fails. Do not edit source code during baseline validation.

### Step 3 — Build the application

Run:

npm run build

Note: this runs electron-vite build.

The Linux packaging script is npm run package:linux. Do not package yet unless separately approved.

### Step 4 — Start development mode

Run:

npm run dev

Verify:
- The Electron window opens.
- The renderer loads.
- The terminal has no blocking runtime errors.
- The window closes normally.

Stop the process with Ctrl+C after testing.

### Step 5 — Record resource usage

Use Ubuntu System Monitor or htop.

Record:
- Idle CPU after the app settles.
- Approximate RAM usage.
- CPU/RAM during basic UI interaction.
- Whether other applications were running.

Do not invent or assume performance targets.

## Do Not Do Yet

- Do not modify Keet source code.
- Do not integrate Gemini or other providers.
- Do not add API keys.
- Do not change PipeWire/PulseAudio settings.
- Do not create virtual sinks or loopbacks.
- Do not apply forced dependency upgrades.
- Do not implement mechanisms to evade interview monitoring.

## Definition of Done

- npm audit findings captured and classified.
- TypeScript check result recorded.
- Build result recorded.
- Development startup result recorded.
- Initial CPU/RAM observations recorded.
- PROJECT_STATUS.md updated.

## Instructions for AI Collaborators

Before proposing code changes, read:

1. PROJECT_CONTEXT.md
2. TECHNICAL_DECISIONS.md
3. PROJECT_STATUS.md
4. NEXT_TASK.md

Only propose actions within the active task.

Clearly distinguish observed facts from assumptions. Never claim to have inspected local files without actual access. Ask for approval before source-code or audio-configuration changes.
