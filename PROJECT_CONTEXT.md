# Keet Interview Copilot — Project Context

Last updated: 2026-11-13

## 1. Project Overview

We are evaluating and adapting the open-source Keet desktop application as the foundation for a personal AI interview copilot.

- Upstream: https://github.com/eeshsaxena/keet
- Local source repository: ~/keet-copilot
- Documentation repository: https://github.com/okihusni/keet-copilot-docs
- Upstream package: keet, version 0.1.0
- Declared license: MIT (verify LICENSE before redistribution)

The objective is to build a reliable desktop assistant for interview preparation and, where permitted by interview rules, live interview assistance.

## 2. Product Goals

1. Capture permitted meeting audio.
2. Keep system audio and microphone input distinguishable.
3. Transcribe speech with low latency and good accuracy.
4. Detect interviewer questions and follow-ups.
5. Generate concise, natural suggested answers.
6. Ground answers in the user's actual CV, experience, job description, and technical notes.
7. Provide a usable desktop interface for transcripts and suggestions.
8. Keep the application lightweight on the target laptop.
9. Prefer cloud inference over local LLM/STT workloads.
10. Keep AI providers replaceable through clear interfaces.

## 3. User Background for Answer Grounding

The user has approximately nine years of QA experience:

- Around four years in game testing.
- More than five years as a QA Engineer.
- Around three years of test automation.
- Tools: Playwright, TypeScript, Cypress, Postman, and SQL.
- Built an automation framework containing more than 500 automated tests.

Interview suggestions must not invent skills, experience, projects, or achievements.

The user's English level is approximately B1 and they are working toward B2. Suggested spoken answers should sound natural and avoid unnecessarily complex vocabulary.

## 4. Target Environment

User-provided hardware and environment:

- OS: Ubuntu Linux
- Desktop: GNOME
- Display server: X11
- CPU: AMD Ryzen 3 3250U
- RAM: 10 GB
- GPU: AMD Radeon Vega integrated graphics
- Swap: 4 GB
- Audio: PipeWire 1.0.5 with PulseAudio compatibility
- Node.js: v22.23.4
- npm: 10.9.9
- Git: 2.43.0

Detected default audio devices:

- Output: alsa_output.pci-0000_05_00.6.analog-stereo
- Input: alsa_input.pci-0000_05_00.6.analog-stereo

Audio devices were reported as SUSPENDED during an idle check. This is not by itself evidence of an audio problem.

## 5. Candidate Technical Direction

Keet is the current foundation candidate, pending successful baseline validation.

Potential architecture to investigate:

- Desktop: Electron, React, TypeScript.
- Audio: Linux/PipeWire-compatible capture.
- Speech-to-text: evaluate a cloud streaming transcription provider.
- Answer generation: separate cloud text-generation provider.
- Retrieval: local knowledge base containing relevant user-approved documents.
- Secrets: keep API keys out of renderer code and source control.
- IPC: typed and validated communication between Electron main and renderer processes.

Gemini Live and Gemini Flash are candidates to investigate, not finalized technology decisions.

Verify exact model IDs, availability, quotas, pricing, latency, and data terms using current official documentation before implementation.

## 6. Privacy and Safety

- Never commit API keys, credentials, CV files, recordings, or private transcripts.
- Review provider data-use and retention terms before sending personal information.
- Send only relevant knowledge-base excerpts to AI providers.
- Obtain appropriate consent before capturing or processing other participants' audio.
- Follow interview and meeting platform rules.
- Do not build mechanisms to evade monitoring or bypass platform controls.
- Do not depend on screen-share exclusion; it is not reliably guaranteed on Linux.

## 7. Current Scope

We are validating the original Keet application before modifying source code.

See:
- TECHNICAL_DECISIONS.md
- PROJECT_STATUS.md
- NEXT_TASK.md
