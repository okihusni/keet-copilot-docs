# Decision Record — Gemini Live Transcript Finalization

**Status:** Open — no implementation decision recorded  
**Owner:** Project owner  
**Date:** To be filled in  
**Source revision:** To be filled in

## Problem statement
In the observed Windows Gemini Live Test B, server input-transcription text arrived after local activity ended, but no serverContent.turnComplete was observed. The manager waited approximately 10 seconds and discarded the text; no final transcript UI event was emitted and the LLM was not called.

The inputTranscription.finished field was not recorded, so its value and semantics remain unknown.

## Evidence baseline
Record exact log paths, timestamps, source commit, test output, and environment. This summary is not a substitute for raw evidence.

## Independent reviews
- **Sonnet:** Pending
- **ChatGPT:** Pending

## Agreements
To be filled after both independent reviews are complete.

## Disagreements and evidence
| Topic | Sonnet position | ChatGPT position | Evidence / resolution |
|---|---|---|---|
| | | | |

## Decision
No code change is approved by this template. Record the accepted approach only after review and owner approval.
- **Chosen approach:**
- **Rationale:**
- **Rejected alternatives:**
- **Known residual risks:**
- **Owner approval / date:**

## Acceptance criteria
Complete before implementation. Include expected event ordering, finalization behavior, duplicate/late event handling, timeout behavior, cleanup, and regression coverage.

## Implementation reference
- **Branch / commit:**
- **Changed files:**
- **Tests and results:**
- **Not run / limitations:**
- **Implementation review outcome:**
