<!--
SYNC IMPACT REPORT
==================
Version change: (template) → 1.0.0 (initial ratification, tech stack included)
Modified principles: N/A (first authoring)
Added sections:
  - Core Principles (5 principles)
  - Technology Stack
  - Development Workflow
  - Governance
Removed sections: N/A
Templates reviewed:
  ✅ .specify/templates/plan-template.md — Constitution Check section aligns with principles
  ✅ .specify/templates/spec-template.md — User story + requirements structure matches principles
  ✅ .specify/templates/tasks-template.md — Task phases align with simplicity/testability principles
  ✅ .specify/templates/agent-file-template.md — Generic, no outdated agent-specific references
Deferred TODOs:
  - None
-->

# Job Application Assistant Constitution

## Core Principles

### I. Agent-Driven Processing

The system's core value is delivered through an AI agent pipeline that orchestrates document
analysis, gap identification, content generation, and skill recommendation.

- Every major capability (gap analysis, cover letter generation, skill suggestions) MUST be
  implemented as a discrete, composable agent step.
- Agent steps MUST be independently executable and testable in isolation.
- Prompt logic MUST be version-controlled alongside code; ad-hoc prompt strings scattered
  across modules are NOT acceptable.

**Rationale**: Discrete steps make debugging tractable, enable partial re-runs, and allow
individual steps to be improved or replaced without destabilizing the pipeline.

### II. Privacy & Data Sensitivity

Resumes and job descriptions contain personally identifiable and professionally sensitive
information and MUST be handled accordingly.

- No user document content (resume text, job description text) MUST be persisted to external
  storage without explicit user consent at runtime.
- Logging MUST NOT capture raw document contents; sanitized summaries or metadata only.
- Third-party API calls (e.g., LLM providers) MUST be documented in user-facing docs so users
  understand where their data travels.

**Rationale**: Trust is foundational for a tool that handles sensitive career data. Violating
privacy erodes user confidence and may create legal exposure.

### III. User Control & Review

The system generates suggestions, not final decisions. Humans MUST remain in control.

- All AI-generated outputs (cover letters, gap analyses, skill suggestions) MUST be presented
  as drafts for user review, never silently applied.
- The system MUST provide clear explanations alongside each recommendation so users can
  evaluate and override them.
- Any destructive or irreversible action (e.g., overwriting a file) MUST require explicit
  user confirmation.

**Rationale**: AI outputs can be incorrect or misaligned with user intent. Surfacing reasoning
and requiring review prevents blind reliance on potentially flawed suggestions.

### IV. Simplicity & YAGNI

Build the minimum necessary to deliver value; avoid speculative features and premature
abstractions.

- Features MUST NOT be added without a concrete user story requiring them.
- Abstractions are justified only when the same pattern appears at least three times.
- Dependencies MUST be evaluated for necessity; each new dependency requires a written
  justification in the relevant plan.md.

**Rationale**: A focused tool is easier to maintain, explain to users, and extend correctly
when real requirements emerge.

### V. Testability

Every component MUST be independently testable without requiring a live LLM call.

- Business logic (gap analysis algorithms, text transformations, file parsing) MUST be
  unit-testable with deterministic inputs and outputs.
- Integration tests MUST use recorded/stubbed LLM responses (fixtures) to avoid
  non-determinism and cost.
- A feature is NOT considered complete until its acceptance scenarios from spec.md pass.

**Rationale**: LLM calls are slow, expensive, and non-deterministic. Isolating logic ensures
fast, reliable CI and prevents regressions when prompts or models change.

## Technology Stack

| Concern | Choice | Notes |
|---------|--------|-------|
| Language | C# / .NET 10 | Target `net10.0`; use `async`/`await` throughout (LLM calls are I/O-bound) |
| AI Framework | Semantic Kernel (Microsoft) | Official agentic SDK; use SK's kernel, plugins, and planner primitives |
| LLM Provider | Google AI Studio — Gemini (free tier) | Accessed via Semantic Kernel's Google connector; API key in user secrets |
| UI | Console application | `System.Console` I/O for MVP; no GUI framework dependency |
| Testing | xUnit + NSubstitute | Unit tests via xUnit; mock SK interfaces with NSubstitute to avoid live LLM calls |
| Logging | `Microsoft.Extensions.Logging` | Structured logging; raw document content MUST NOT appear in log output |

**Constraints**:

- All new NuGet dependencies MUST be justified in the relevant `plan.md` before being added.
- Semantic Kernel plugin/function boundaries define the composable agent steps required by
  Principle I; logic MUST NOT bypass the SK abstraction by calling the Gemini HTTP API directly.
- The Gemini free-tier rate limits (requests/minute, tokens/day) MUST be accounted for in
  integration test design — tests MUST NOT rely on live API calls in CI.

## Development Workflow

- All work MUST occur on feature branches named `###-short-description` (e.g., `001-resume-parser`).
- Pull requests MUST reference the relevant `spec.md` and pass all automated tests before merge.
- The Constitution Check section in each `plan.md` MUST be completed and signed off before
  implementation begins.
- Complexity violations (deviations from Principle IV) MUST be documented in the
  Complexity Tracking table of the relevant `plan.md`.

## Governance

This constitution supersedes all other development practices. Any practice not addressed here
defaults to the principle of Simplicity (Principle IV).

**Amendment procedure**:
1. Author a proposal describing the change and rationale.
2. Increment `CONSTITUTION_VERSION` per semantic versioning rules (see below).
3. Update `LAST_AMENDED_DATE` to the amendment date.
4. Propagate changes to all dependent templates and runtime guidance files.
5. Record the change in the Sync Impact Report comment at the top of this file.

**Versioning policy**:
- MAJOR: Removal or redefinition of an existing principle (backward-incompatible governance change).
- MINOR: New principle or section added, or material guidance expanded.
- PATCH: Clarifications, wording corrections, or non-semantic refinements.

**Compliance review**: Each pull request implicitly reviews compliance. A dedicated governance
review SHOULD occur when three or more features have shipped since the last amendment.

**Version**: 1.0.0 | **Ratified**: 2026-03-10 | **Last Amended**: 2026-03-10
