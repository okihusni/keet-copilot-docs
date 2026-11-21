# AI & Human Collaboration Protocol

**Status:** Proposed for review  
**Owner:** Project owner  
**Applies to:** Keet Copilot documentation and engineering work coordinated through this repository

## 1. Purpose
Establish a repeatable, evidence-led workflow for the project owner, ChatGPT, Claude/Sonnet, and Antigravity. Decisions must be reviewable, disagreements explicit, and authority boundaries clear.

This protocol does not grant access to the owner's local machine, authorize real API usage, or authorize application-code changes by itself.

## 2. Source of truth
1. The application source repository is authoritative for application code.
2. This repository is authoritative for shared project notes, review artifacts, decisions, and handoff documentation.
3. The current branch and pull request are authoritative for a proposed documentation change until merged.
4. Local workspace state must be verified by the person/tool operating in that workspace. A conversation summary is not proof of local state.
5. Record relevant commit hashes, file paths, commands, environment details, and timestamps.

## 3. Evidence and claim labels
Use these labels in technical reports:
- **Verified** — directly observed or reproduced, with evidence.
- **Observed** — seen in a limited run; scope and limitations stated.
- **Hypothesis** — plausible explanation that still needs validation.
- **Proposal** — suggested change, not yet approved.
- **Decision** — explicitly accepted by the project owner or agreed process.
- **Blocked / Unknown** — required evidence or access is unavailable.

Never present a hypothesis as a root cause or a proposal as implemented. Include negative results and missing telemetry when relevant.

## 4. Review workflow
1. **Define the question.** Record scope, constraints, expected behavior, and acceptance criteria.
2. **Capture a baseline.** Record current behavior, relevant code paths, logs, tests, environment, and known limitations.
3. **Independent first-pass reviews.** ChatGPT and Sonnet review the same scoped evidence independently, before reading each other's conclusions.
4. **Challenge with evidence.** Compare findings and challenge claims using code references, logs, tests, or documented platform behavior. Do not manufacture disagreement.
5. **Resolve.** Prepare a decision record. The project owner resolves material trade-offs or risks that remain disputed. Unresolved items stay open.
6. **Approve implementation scope.** Only accepted decisions become implementation tasks. Record in-scope and out-of-scope work.
7. **Implement in a controlled workspace.** Antigravity (or the designated implementer) makes only approved changes and reports exact files changed.
8. **Verify.** Run agreed tests and checks. Record commands, outcomes, environment, and relevant output. A test not run is not passed.
9. **Review implementation.** ChatGPT and Sonnet inspect the diff and evidence against the decision and acceptance criteria.
10. **Owner acceptance.** The project owner explicitly approves merge, release, packaging, or real-world tests requiring authorization.

For small, low-risk documentation edits, the owner may choose a lighter review and record that choice in the PR.

## 5. Required review report format
Every substantive review includes:
- Scope and exact revision/commit reviewed
- Evidence inspected (paths, line references, logs, test output)
- Findings ordered by severity
- For each finding: impact, reproduction/reasoning, evidence, confidence, and recommended action
- What was not inspected or could not be verified
- Questions or decisions required from the owner

Severity guidance:
- **Critical:** security, privacy, data-loss, or application-integrity risk requiring immediate stop.
- **High:** major functional failure, likely regression, or serious reliability risk.
- **Medium:** meaningful defect or missing coverage with a workaround.
- **Low:** minor defect, maintainability concern, or documentation issue.
- **Informational:** observation or improvement suggestion.

Avoid severity inflation. Style preferences are not defects.

## 6. Decision rules
- Correctness, safety, privacy, and data integrity take priority over speed.
- Prefer the smallest change that satisfies the accepted requirement.
- Preserve platform separation and provider boundaries.
- Do not change unrelated behavior to make a test pass.
- Passing tests provide evidence only for covered behavior.
- If reviewers disagree, compare assumptions and evidence; record residual risk and the owner decision.
- The project owner has final authority over product behavior, scope, risk acceptance, and release.

## 7. Safety and authorization boundaries
Explicit project-owner approval is required before:
- Real external API tests that may incur cost, consume quota, transmit sensitive data, or affect live services
- Packaging, signing, distributing, or replacing an application executable
- Merging a pull request or releasing/deploying changes
- Destructive or hard-to-reverse local Git/filesystem operations (reset, clean, force push, deleting user data, discarding unrelated changes)
- Changing microphone/system-audio capture scope, privacy controls, or content-protection behavior
- Any change that exposes secrets, credentials, recordings, transcripts, or personal data

Never commit API keys, access tokens, credentials, private recordings, raw personal transcripts, or sensitive environment files. Redact secrets from logs.

Repository authorization does not grant access to local files or permission to run external services.

## 8. Test and implementation evidence
For each implementation task, report:
- Branch/commit and changed paths
- Tests/checks run and exact results
- Tests/checks not run and why
- Manual test steps and observed results, if applicable
- Environment and dependency versions when relevant
- Known risks, limitations, and rollback considerations

Do not claim a build, test, API call, or manual verification succeeded unless its result was observed.

## 9. Pull request hygiene
- Use a focused branch and descriptive PR.
- Explain reason, scope, evidence, and validation.
- Keep unrelated platform documentation and application code untouched.
- Do not merge a draft or proposed change without explicit owner approval.
- Update decision/review records when implementation differs materially from the accepted proposal.

## 10. Pilot workflow
Use the Gemini Live transcript-finalization investigation as the first pilot. The initial review is limited to the evidence and current behavior described in its review records. Do not infer missing event fields or unobserved behavior. This documentation task does not authorize a new real API test, packaging, or application-source modification.

## 11. Change control
This protocol is proposed until accepted by the project owner. Amend it through a pull request and record material changes in the PR description. If a rule conflicts with an explicit owner instruction or platform/security requirement, stop and clarify.
